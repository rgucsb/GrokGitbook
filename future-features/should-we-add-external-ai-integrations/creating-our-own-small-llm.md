# Creating our Own Small-LLM

Let’s create a plan to develop and train a **small Language Learning Model (LLM)** for the **SAT Prep Suite** to minimize costs, especially as the student base and data grow. This approach aims to reduce reliance on expensive external APIs (e.g., GPT-4o) while maintaining high-quality, personalized feedback and explanations. By leveraging the suite’s existing data (responses, questions, lessons) and scaling with new student interactions, we can build a cost-effective, domain-specific model tailored to SAT prep needs as of March 26, 2025.

***

#### Step 1: Rationale and Feasibility

**Why Develop Our Own LLM?**

* **Cost Reduction**: External LLM APIs (e.g., GPT-4o at \~$0.005/1K tokens) scale linearly with usage. With thousands of students, costs could exceed $100s/month. A custom model has a one-time training cost and low inference costs.
* **Data Availability**: The suite already has response patterns, question rationales, and lesson content—ideal for training a domain-specific model. More students = more data.
* **Customization**: A small LLM can focus on SAT-specific language, skills, and tutoring tone, avoiding the overhead of general-purpose models.
* **Scalability**: Self-hosted inference scales with infrastructure, not API quotas.

**Feasibility**

* **Size**: A small LLM (e.g., 100M-500M parameters) is trainable on modest hardware (e.g., a single GPU like NVIDIA A100) and efficient for inference on CPUs or edge devices.
* **Existing Tools**: Open-source frameworks (e.g., Hugging Face Transformers, PyTorch) and pre-trained models (e.g., DistilBERT, TinyLLaMA) reduce development time.
* **Data**: Thousands of responses and growing student interactions provide a rich dataset for fine-tuning.

***

#### Step 2: Proposed Plan

**Phase 1: Model Selection and Data Preparation**

* **Duration**: 2-3 weeks
* **Steps**:
  1. **Choose Base Model**:
     * **Options**: DistilBERT (66M params), TinyLLaMA (300M params), or a custom transformer.
     * **Why**: Small, efficient, and proven for educational tasks. TinyLLaMA offers a good balance of size and performance.
  2. **Collect Data**:
     * **Sources**: `RESPONSES` (answers, time spent), `QUESTIONS` (content, rationales), `LESSONS` (text), `STUDY_PLANS` (tasks), and user feedback logs.
     * **Format**: Convert to text pairs (e.g., “Question: Solve 2x + 3 = 7” → “Answer: Subtract 3, then divide by 2; x = 2”).
     * **Initial Size**: Assume 10K student responses + 1K questions/lessons ≈ 50K text pairs.
  3. **Preprocess Data**:
     * Clean text (remove noise, standardize format).
     * Augment with synthetic data (e.g., rephrase rationales using templates).
     * Split: 80% train, 10% validation, 10% test.

**Phase 2: Model Training**

* **Duration**: 3-4 weeks
* **Steps**:
  1. **Hardware**:
     * **Training**: Rent 1 NVIDIA A100 GPU (40GB) via AWS/GCP (\~$3/hr, \~$500 for 1 week).
     * **Inference**: Use CPU (e.g., AWS EC2 c6i.large, \~$0.10/hr) post-training.
  2. **Training Setup**:
     * **Framework**: PyTorch + Hugging Face Transformers.
     * **Objective**: Fine-tune on SAT-specific tasks (feedback generation, question explanation).
     * **Hyperparameters**: Batch size 16, learning rate 2e-5, epochs 3-5.
     * **Loss**: Cross-entropy for next-token prediction.
  3. **Process**:
     * Start with pre-trained weights (e.g., TinyLLaMA).
     * Fine-tune on SAT dataset using supervised learning.
     * Monitor validation loss; stop if overfitting occurs.
  4. **Evaluation**:
     * Metrics: BLEU/ROUGE (text similarity), perplexity (<20 target), human review (tutor-like tone).
     * Compare to GPT-4o outputs on 100 sample inputs.

**Phase 3: Integration and Deployment**

* **Duration**: 2-3 weeks
* **Steps**:
  1. **Backend Integration**:
     * Replace GPT-4o calls with local model inference in `api/utils.py`.
     * Use ONNX or TorchScript for optimized runtime.
  2. **Inference Setup**:
     * Deploy on a Flask endpoint (e.g., `/llm/feedback`) with model loaded in memory.
     * Cache responses in Redis for frequent queries.
  3. **Mobile App**:
     * Keep API-based calls; no client-side changes needed.
  4. **Testing**:
     * A/B test custom LLM vs. GPT-4o with 500 students (feedback quality, response time).
     * Monitor latency (<1s target) and accuracy.

**Phase 4: Scaling and Maintenance**

* **Duration**: Ongoing
* **Steps**:
  1. **Data Growth**:
     * As student base grows (e.g., 10K → 100K), retrain quarterly with new responses (100K+ text pairs).
     * Incremental fine-tuning to avoid retraining from scratch.
  2. **Infrastructure**:
     * Scale inference servers (e.g., 2-5 EC2 instances) based on load.
     * Use load balancer for high traffic.
  3. **Cost Management**:
     * Monitor compute costs; optimize with quantization (e.g., 8-bit inference).

***

#### Step 3: Technical Implementation

**Training Script (`train_llm.py`)**

```python
from transformers import AutoModelForCausalLM, AutoTokenizer, Trainer, TrainingArguments
import torch
from datasets import load_dataset

# Load pre-trained model and tokenizer
model_name = "TinyLLaMA-300M"  # Hypothetical small model
model = AutoModelForCausalLM.from_pretrained(model_name)
tokenizer = AutoTokenizer.from_pretrained(model_name)

# Prepare dataset
dataset = load_dataset("json", data_files="sat_data.json")
def tokenize_function(examples):
    return tokenizer(examples["input"], examples["output"], truncation=True, padding="max_length", max_length=128)

tokenized_dataset = dataset.map(tokenize_function, batched=True)

# Training arguments
training_args = TrainingArguments(
    output_dir="./sat_llm",
    num_train_epochs=3,
    per_device_train_batch_size=16,
    per_device_eval_batch_size=16,
    warmup_steps=500,
    weight_decay=0.01,
    logging_dir="./logs",
    logging_steps=10,
    evaluation_strategy="epoch",
    save_strategy="epoch",
    load_best_model_at_end=True,
)

# Trainer
trainer = Trainer(
    model=model,
    args=training_args,
    train_dataset=tokenized_dataset["train"],
    eval_dataset=tokenized_dataset["validation"],
)

# Train
trainer.train()
model.save_pretrained("./sat_llm_final")
tokenizer.save_pretrained("./sat_llm_final")
```

**`api/utils.py` (Updated with Custom LLM)**

```python
import torch
from transformers import AutoModelForCausalLM, AutoTokenizer
import redis
import json
from typing import Dict

# Load custom LLM
model = AutoModelForCausalLM.from_pretrained("./sat_llm_final")
tokenizer = AutoTokenizer.from_pretrained("./sat_llm_final")
model.eval()
redis_client = redis.Redis(host=os.getenv("REDIS_HOST"), port=int(os.getenv("REDIS_PORT")), decode_responses=True)

def generate_feedback_with_custom_llm(response_patterns: Dict, predictions: Dict) -> str:
    """Use custom LLM instead of GPT-4o"""
    cache_key = f"llm_feedback_{hash(json.dumps(response_patterns))}_{hash(json.dumps(predictions))}"
    cached = redis_client.get(cache_key)
    if cached:
        return cached
    
    prompt = (
        "You’re a friendly SAT tutor. Analyze this data: "
        f"Performance: {json.dumps(response_patterns)}. Predictions: {json.dumps(predictions)}. "
        "Write a 150-200 word feedback message with strengths, weaknesses, and steps."
    )
    inputs = tokenizer(prompt, return_tensors="pt", max_length=128, truncation=True)
    with torch.no_grad():
        outputs = model.generate(**inputs, max_new_tokens=250, temperature=0.7, top_p=0.9)
    feedback = tokenizer.decode(outputs[0], skip_special_tokens=True).split(prompt)[-1].strip()
    
    redis_client.setex(cache_key, 86400, feedback)
    return feedback

# Replace GPT-4o in existing functions
def enhance_feedback_with_gpt(response_patterns: Dict, predictions: Dict) -> str:
    return generate_feedback_with_custom_llm(response_patterns, predictions)

# Keep other integrations (Knewton, Watson, Speech) unchanged
```

**`api/routes/review.py` (Updated)**

```python
@router.get("/next/{user_id}")
async def get_next_steps(user_id: str, db: Session = Depends(get_db)):
    analysis = analyze_response_patterns(db, user_id)
    predictions = predict_skill_growth_with_watson(db, user_id, analysis["skill_history"])
    feedback = generate_feedback_with_custom_llm(analysis["response_patterns"], predictions)
    return {"user_id": user_id, "skill_predictions": predictions, "custom_feedback": feedback}
```

***

#### Step 4: Cost Analysis

**Initial Costs**

* **Training**:
  * 1 A100 GPU × 168 hours (1 week) × $3/hr = \~$500.
  * Data prep (engineer time): \~$1,000 (20 hours × $50/hr).
  * **Total**: \~$1,500 one-time.
* **Inference**:
  * 1 EC2 c6i.large (\~$0.10/hr) × 720 hours/month = \~$72/month.
  * Scales to \~$360/month for 5 instances at 100K students.

**Comparison to GPT-4o**

* **GPT-4o**:
  * 10K students × 10 feedbacks/month × $0.01 = $1,000/month.
  * 100K students = $10,000/month.
* **Custom LLM**:
  * Initial $1,500 + $72-$360/month = Breaks even at \~2K feedbacks/month (\~200 students).

**Long-Term Savings**

* At 10K students: $1,000 (GPT-4o) vs. $150 (custom LLM) = $850/month saved.
* At 100K students: $10,000 vs. $360 = $9,640/month saved.

***

#### Step 5: Benefits and Trade-offs

**Benefits**

* **Cost Efficiency**: Scales with users without API fees.
* **Domain Specificity**: Tailored to SAT vocab, skills, and tutoring style.
* **Control**: Full ownership, no dependency on third-party uptime/pricing.
* **Data Leverage**: Improves with more student data.

**Trade-offs**

* **Upfront Effort**: 2-3 months to develop vs. instant API use.
* **Performance**: May lack GPT-4o’s general knowledge; limited to SAT domain.
* **Maintenance**: Requires retraining and server management.

***

#### Step 6: Timeline and Next Steps

* **Month 1**: Select model, prep data.
* **Month 2**: Train and evaluate.
* **Month 3**: Integrate, deploy, A/B test vs. GPT-4o.
* **Ongoing**: Retrain quarterly with new data.

**Next Steps**

* **Pilot**: Train on current data (10K responses) and test with 100 students.
* **Fallback**: Retain GPT-4o as backup during transition.
* **Expand**: Add ACT-specific fine-tuning later.

***

#### Conclusion

Developing a small LLM (e.g., 300M params) is viable and cost-effective for the SAT Prep Suite as student numbers grow. With an initial investment of \~$1,500 and monthly costs of $72-$360, it saves thousands compared to GPT-4o at scale, leveraging existing data for a tailored solution by Q3 2025. Ready to proceed with a pilot? Let me know your thoughts!
