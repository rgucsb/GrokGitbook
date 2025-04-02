# Code

The latest code is a robust tool for extracting SAT Math questions from a PDF and structuring them into a JSON format, with the added functionality of assigning IRT 3PL parameters. It supports a wide range of question types, handles visual elements like graphs and tables, and provides detailed metadata and rationales. The IRT 3PL parameters enable psychometric analysis, making the output suitable for educational research, test development, or adaptive learning systems. With some potential improvements, the code can be made even more accurate and scalable for real-world applications.

#### How to Run the Code

To run the code on your local machine:

1. **Install Dependencies**:
   * Install the required Python packages:bashCollapseWrapCopy`pip install opencv-python pytesseract pillow pdf2image`
   * Install Tesseract OCR and ensure it's in your system path (e.g., on Windows, install it and add to PATH; on Linux, use sudo apt-get install tesseract-ocr).
   * Install Poppler for pdf2image (e.g., on Windows, download and add to PATH; on Linux, use sudo apt-get install poppler-utils).
2. **Prepare the PDF**:
   * Save the PDF file as questions.pdf in the same directory as the script.
3. **Run the Script**:
   * Execute the script, which will process the first 15 pages and save the output to output\_questions.json.
4. **Check the Output**:
   * The JSON file will contain the structured data as shown above, with actual base64 strings for tables instead of placeholders.



#### Updated Code with IRT 3PL Parameters

```python
import cv2
import numpy as np
from PIL import Image
import pytesseract
import base64
import json
import io
import os
from pdf2image import convert_from_path
import re
import logging

# Set up logging
logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(levelname)s - %(message)s')

def image_to_base64(image):
    """Convert an image to base64 string."""
    buffered = io.BytesIO()
    Image.fromarray(image).save(buffered, format="PNG")
    return base64.b64encode(buffered.getvalue()).decode('utf-8')

def detect_equations(text):
    """Detect mathematical equations in the text and return a list of equations."""
    latex_pattern = r'\$(.*?)\$|\$\$(.*?)\$\$'
    math_symbol_pattern = r'[\w\s]*[+\-*/=^]+[\w\s]*|\w+\s*=\s*[\w\d\s+\-*/^]+'
    
    equations = []
    
    # Find LaTeX equations
    latex_matches = re.finditer(latex_pattern, text)
    for match in latex_matches:
        equation = match.group(1) if match.group(1) else match.group(2)
        if equation:
            equations.append(equation)
    
    # Find inline equations with math symbols
    lines = text.split('\n')
    for line in lines:
        if re.search(math_symbol_pattern, line):
            equation_part = re.search(math_symbol_pattern, line)
            if equation_part:
                equation = equation_part.group(0).strip()
                if equation and equation not in equations:
                    equations.append(equation)
    
    return equations

def detect_lines(image):
    """Detect straight lines in the image using Hough Line Transform."""
    gray = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)
    edges = cv2.Canny(gray, 50, 150, apertureSize=3)
    lines = cv2.HoughLinesP(edges, 1, np.pi / 180, threshold=100, minLineLength=100, maxLineGap=10)
    return lines if lines is not None else []

def detect_curves(image):
    """Detect curves by analyzing contours for non-rectangular shapes."""
    gray = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)
    edges = cv2.Canny(gray, 50, 150, apertureSize=3)
    contours, _ = cv2.findContours(edges, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)
    
    curve_count = 0
    for contour in contours:
        epsilon = 0.02 * cv2.arcLength(contour, True)
        approx = cv2.approxPolyDP(contour, epsilon, True)
        if len(approx) > 4:
            curve_count += 1
    return curve_count

def detect_scatter_points(image):
    """Detect scatter points by identifying small, isolated contours."""
    gray = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)
    edges = cv2.Canny(gray, 50, 150, apertureSize=3)
    contours, _ = cv2.findContours(edges, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)
    
    scatter_count = 0
    for contour in contours:
        area = cv2.contourArea(contour)
        if 10 < area < 100:
            scatter_count += 1
    return scatter_count

def detect_graph_or_table(image):
    """Detect if there's a graph or table in the image and classify it."""
    gray = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)
    edges = cv2.Canny(gray, 50, 150, apertureSize=3)
    contours, _ = cv2.findContours(edges, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)
    
    for contour in contours:
        x, y, w, h = cv2.boundingRect(contour)
        if w * h > 0.1 * image.shape[0] * image.shape[1]:
            region = image[y:y+h, x:x+w]
            
            lines = detect_lines(region)
            num_lines = len(lines)
            
            num_curves = detect_curves(region)
            num_scatter_points = detect_scatter_points(region)
            
            if num_lines > 5:
                horizontal_lines = 0
                vertical_lines = 0
                for line in lines:
                    x1, y1, x2, y2 = line[0]
                    if abs(x1 - x2) > abs(y1 - y2):
                        horizontal_lines += 1
                    else:
                        vertical_lines += 1
                if horizontal_lines > 2 and vertical_lines > 2:
                    return True, (x, y, w, h), "table"
            
            if num_curves > 2 or num_scatter_points > 10:
                return True, (x, y, w, h), "graph"
            
            if num_lines > 0:
                return True, (x, y, w, h), "graph"
            
            return True, (x, y, w, h), "table"
    
    return False, None, None

def extract_text(image, exclude_region=None):
    """Extract text from an image, optionally excluding a region."""
    if exclude_region:
        x, y, w, h = exclude_region
        image[y:y+h, x:x+w] = 255
    text = pytesseract.image_to_string(image)
    return text.strip()

def parse_metadata(text):
    """Parse metadata from the text (Question ID, Assessment, Test, Domain, Skill, Difficulty)."""
    metadata = {
        "Question ID": "",
        "Assessment": "",
        "Test": "",
        "Domain": "",
        "Skill": "",
        "Difficulty": ""
    }
    
    question_id_match = re.search(r"Question ID\s*([a-f0-9]+)", text)
    if question_id_match:
        metadata["Question ID"] = question_id_match.group(1)
    
    lines = text.split('\n')
    for line in lines:
        if '|' in line:
            parts = [part.strip() for part in line.split('|')]
            if len(parts) >= 6:
                if parts[0] == "Assessment":
                    continue
                metadata["Assessment"] = parts[0]
                metadata["Test"] = parts[1]
                metadata["Domain"] = parts[2]
                metadata["Skill"] = parts[3]
                metadata["Difficulty"] = parts[4]
    
    return metadata

def extract_answer_choices(lines, page_num):
    """Extract answer choices and validate for MCQs."""
    answer_choices = []
    answer_choice_pattern = r'^(?:[a-dA-D][.)]\s*.*|[A-D]\s*.*)$'
    
    # First pass: Collect potential answer choices
    for line in lines:
        line = line.strip()
        if not line:
            continue
        if re.match(answer_choice_pattern, line):
            # Normalize the label (e.g., 'a.' to 'A.')
            label = line[0].upper()
            if label in ['A', 'B', 'C', 'D']:
                answer_choices.append(line)
    
    # Validate and recover if necessary
    if answer_choices:
        # Ensure exactly 4 answer choices for MCQs
        if len(answer_choices) < 4:
            logging.warning(f"Page {page_num}: Found only {len(answer_choices)} answer choices, expected 4. Attempting to recover...")
            # Second pass: Look for missing choices in the remaining lines
            expected_labels = ['A', 'B', 'C', 'D']
            found_labels = [choice[0].upper() for choice in answer_choices]
            missing_labels = [label for label in expected_labels if label not in found_labels]
            
            for line in lines:
                line = line.strip()
                if not line:
                    continue
                for label in missing_labels:
                    if line.lower().startswith(label.lower()) or f"{label}." in line or f"{label})" in line:
                        answer_choices.append(f"{label}. {line}")
                        missing_labels.remove(label)
                        break
            
            # If still missing, log an error
            if len(answer_choices) < 4:
                logging.error(f"Page {page_num}: Could not recover all answer choices. Found: {answer_choices}")
    
    # Sort answer choices by label (A, B, C, D)
    if answer_choices:
        answer_choices.sort(key=lambda x: x[0].upper())
    
    return answer_choices

def assign_irt_parameters(difficulty, question_type):
    """
    Assign IRT 3PL parameters based on difficulty level and question type.
    
    Parameters:
    - difficulty (str): 'Easy', 'Medium', or 'Hard'
    - question_type (str): 'MCQ' or 'FRQ'
    
    Returns:
    - dict: IRT parameters {a, b, c}
    """
    # Discrimination parameter (a)
    a_values = {
        "Easy": 0.8,
        "Medium": 1.0,
        "Hard": 1.2
    }
    
    # Difficulty parameter (b)
    b_values = {
        "Easy": -1.0,
        "Medium": 0.0,
        "Hard": 1.0
    }
    
    # Guessing parameter (c)
    # For MCQs with 4 options, c = 0.25; for FRQs, c = 0.0
    c = 0.25 if question_type == "MCQ" else 0.0
    
    # Assign parameters based on difficulty
    difficulty = difficulty.capitalize()
    if difficulty not in a_values:
        logging.warning(f"Unknown difficulty level '{difficulty}', defaulting to Medium.")
        difficulty = "Medium"
    
    irt_parameters = {
        "a": a_values[difficulty],
        "b": b_values[difficulty],
        "c": c
    }
    
    return irt_parameters

def process_pdf_to_json(pdf_path, output_json_path, start_page=36, end_page=50):
    """Process a PDF file with one question per page and save to JSON, including IRT 3PL parameters."""
    pages = convert_from_path(pdf_path, dpi=300)
    
    results = []
    
    for page_num, page in enumerate(pages[start_page-1:end_page], start_page):
        image = cv2.cvtColor(np.array(page), cv2.COLOR_RGB2BGR)
        
        result = {
            "page": page_num,
            "metadata": {},
            "question": "",
            "graph_or_table": None,
            "equations": [],  # List of LaTeX strings
            "question_type": "",
            "answer_choices": [],
            "correct_answer": "",
            "rationale": "",
            "irt_parameters": {}  # New field for IRT 3PL parameters
        }
        
        has_graph_or_table, boundaries, region_type = detect_graph_or_table(image)
        
        if has_graph_or_table:
            x, y, w, h = boundaries
            region_image = image[y:y+h, x:x+w]
            region_base64 = image_to_base64(region_image)
            result["graph_or_table"] = {
                "type": region_type,
                "base64": region_base64,
                "boundaries": {"x": x, "y": y, "width": w, "height": h}
            }
            text = extract_text(image, exclude_region=boundaries)
        else:
            text = extract_text(image)
        
        result["metadata"] = parse_metadata(text)
        
        # Detect equations (but do not render them as images)
        equations = detect_equations(text)
        result["equations"] = equations  # Store the LaTeX strings directly
        
        lines = text.split('\n')
        question_lines = []
        answer_choices = []
        rationale_lines = []
        in_rationale = False
        correct_answer = ""
        
        for line in lines:
            line = line.strip()
            if not line:
                continue
            
            if "Question ID" in line or "Assessment" in line or "Test" in line or "Domain" in line or "Skill" in line or "Difficulty" in line or line.startswith('|'):
                continue
            
            if "Correct Answer:" in line:
                correct_answer = line.replace("Correct Answer:", "").strip()
                in_rationale = True
                continue
            
            if in_rationale:
                if "Question Difficulty:" in line:
                    continue
                rationale_lines.append(line)
                continue
            
            question_lines.append(line)
        
        # Extract answer choices from the question lines
        answer_choices = extract_answer_choices(question_lines, page_num)
        
        # Remove answer choices from question lines
        answer_choice_pattern = r'^(?:[a-dA-D][.)]\s*.*|[A-D]\s*.*)$'
        question_lines = [line for line in question_lines if not re.match(answer_choice_pattern, line.strip())]
        
        # Join question lines and remove the Question ID prefix
        question_text = " ".join(question_lines).strip()
        # Remove 'ID: <question_id>' from the beginning of the question text
        question_text = re.sub(r'^ID:\s*[a-f0-9]+\s*', '', question_text).strip()
        
        result["question"] = question_text
        result["answer_choices"] = answer_choices
        result["correct_answer"] = correct_answer
        result["rationale"] = " ".join(rationale_lines).strip()
        
        # Determine question type
        result["question_type"] = "MCQ" if answer_choices else "FRQ"
        
        # Validate MCQ: Ensure exactly 4 answer choices
        if result["question_type"] == "MCQ" and len(answer_choices) != 4:
            logging.error(f"Page {page_num}: MCQ has {len(answer_choices)} answer choices, expected 4: {answer_choices}")
        
        # Assign IRT 3PL parameters based on difficulty and question type
        difficulty = result["metadata"].get("Difficulty", "Medium")
        question_type = result["question_type"]
        result["irt_parameters"] = assign_irt_parameters(difficulty, question_type)
        
        results.append(result)
    
    with open(output_json_path, 'w') as f:
        json.dump(results, f, indent=4)
    
    return results

if __name__ == "__main__":
    pdf_path = "questions.pdf"
    output_json_path = "output_questions_36_50.json"
    
    try:
        results = process_pdf_to_json(pdf_path, output_json_path, start_page=36, end_page=50)
        print("Processed successfully. JSON output:")
        print(json.dumps(results, indent=4))
    except Exception as e:
        print(f"Error: {str(e)}")
```

#### Changes Made to the Code

1. **Added `assign_irt_parameters` Function**:
   * This new function takes the difficulty level (e.g., "Easy", "Medium", "Hard") and question type ("MCQ" or "FRQ") as inputs.
   * It assigns the IRT 3PL parameters ((a), (b), (c)) based on the following rules:
     * **Discrimination ((a))**: Easy = 0.8, Medium = 1.0, Hard = 1.2
     * **Difficulty ((b))**: Easy = -1.0, Medium = 0.0, Hard = 1.0
     * **Guessing ((c))**: MCQ = 0.25 (for 4 options), FRQ = 0.0
   * The function returns a dictionary with the parameters.
2. **Updated `result` Dictionary**:
   * Added a new field `"irt_parameters": {}` to the `result` dictionary for each question to store the IRT 3PL parameters.
3. **Integrated IRT Parameter Assignment in `process_pdf_to_json`**:
   * After determining the question type ("MCQ" or "FRQ") and extracting the difficulty from the metadata, the code calls `assign_irt_parameters` to compute the IRT parameters.
   * The computed parameters are stored in the `irt_parameters` field of the `result` dictionary.
4. **Updated JSON Output**:
   * The JSON output now includes the `irt_parameters` field for each question, containing the (a), (b), and (c) values.

#### Explanation of IRT 3PL Parameters

* **(a) (Discrimination)**: Measures how well the item differentiates between test-takers of different abilities. Higher values indicate better discrimination.
* **(b) (Difficulty)**: The ability level ((\theta)) at which a test-taker has a 50% chance of answering correctly (adjusted for guessing). It’s on a standard normal scale.
* **(c) (Guessing)**: The probability of a low-ability test-taker answering correctly by guessing. For MCQs with 4 options, (c = 0.25); for FRQs (grid-in questions), (c = 0.0).

#### Example of Updated JSON Output

If the code processes a page with a Medium difficulty MCQ, the JSON entry might look like this:

```json
{
    "page": 36,
    "metadata": {
        "Question ID": "2t6s6t6s",
        "Assessment": "SAT",
        "Test": "Math",
        "Domain": "Geometry and Trigonometry",
        "Skill": "Lines, angles, and triangles",
        "Difficulty": "Easy"
    },
    "question": "In triangle TUV, angle T = 50 degrees, angle U = 60 degrees. What is the measure of angle V?",
    "graph_or_table": null,
    "equations": [],
    "question_type": "FRQ",
    "answer_choices": [],
    "correct_answer": "70",
    "rationale": "angle V = 180 - 50 - 60 = 70 degrees.",
    "irt_parameters": {
        "a": 0.8,
        "b": -1.0,
        "c": 0.0
    }
}
```

#### Notes

* The code assumes that the difficulty level is extracted correctly from the metadata. If the difficulty field is missing or invalid, it defaults to "Medium" and logs a warning.
* The guessing parameter ((c)) is set based on the question type, which is determined by the presence of answer choices (MCQ) or lack thereof (FRQ).
* This implementation provides a static assignment of IRT parameters. In a real application, these parameters might be estimated using statistical methods (e.g., maximum likelihood estimation) based on actual response data.

Let me know if you need further clarification or additional modifications!
