# Add voice input and conversations

To further enhance the **Advanced AI Tutor** in the **SAT Prep Suite**, I’ll add **voice input** capabilities and make the tutor more **conversational** by improving its natural language understanding and response flow. This builds on the existing enhancements (fine-tuned LLM, real-time chat, predictive analytics) and integrates voice recognition for a hands-free experience, making it more engaging and interactive, akin to a real tutor. I’ll update the React web and React Native mobile apps to support voice input, modify the backend to handle conversational context, and ensure seamless integration as of March 26, 2025. Below, I’ll detail the changes, provide updated code, and outline a testing plan.

***

### Enhancements: Voice Input and Conversational AI Tutor

#### 1. Voice Input

* **Goal**: Allow users to speak questions or commands (e.g., “Explain this question”) instead of typing.
* **Implementation**: Use Web Speech API (web) and `react-native-voice` (mobile) to capture voice input and send it to the AI tutor.

**Web (`web/src/`)**

* Update `practice.js`:

```javascript
import { useRef, useState, useEffect } from 'react';
import { TextField, Button, List, ListItem, Typography, Box, IconButton } from '@mui/material';
import MicIcon from '@mui/icons-material/Mic';
import StopIcon from '@mui/icons-material/Stop';
// ...
const [isRecording, setIsRecording] = useState(false);
const recognitionRef = useRef(null);

useEffect(() => {
  if ('webkitSpeechRecognition' in window) {
    recognitionRef.current = new window.webkitSpeechRecognition();
    recognitionRef.current.continuous = false;
    recognitionRef.current.interimResults = false;
    recognitionRef.current.onresult = (event) => {
      const transcript = event.results[0][0].transcript;
      setChatInput(transcript);
      sendChat(transcript);  // Auto-send voice input
    };
    recognitionRef.current.onend = () => setIsRecording(false);
  }
  // Existing WebSocket setup...
}, [dispatch, mode]);

const toggleRecording = () => {
  if (isRecording) {
    recognitionRef.current.stop();
  } else {
    recognitionRef.current.start();
    setIsRecording(true);
  }
};

const sendChat = (text = chatInput) => {
  if (wsRef.current && text) {
    wsRef.current.send(text);
    setChatMessages((prev) => [...prev, { sender: 'You', text }]);
    setChatInput('');
  }
};
// ...
<Box sx={{ mt: 4 }}>
  <Typography variant="h6">AI Tutor Chat</Typography>
  <List sx={{ maxHeight: 200, overflow: 'auto' }}>
    {chatMessages.map((msg, idx) => (
      <ListItem key={idx}>
        <Typography>{`${msg.sender}: ${msg.text}`}</Typography>
      </ListItem>
    ))}
  </List>
  <Box sx={{ display: 'flex', alignItems: 'center' }}>
    <TextField
      label="Ask a question or use voice"
      value={chatInput}
      onChange={(e) => setChatInput(e.target.value)}
      fullWidth
      onKeyPress={(e) => e.key === 'Enter' && sendChat()}
    />
    <IconButton onClick={toggleRecording} sx={{ ml: 1 }}>
      {isRecording ? <StopIcon /> : <MicIcon />}
    </IconButton>
  </Box>
  <Button variant="contained" onClick={() => sendChat()} sx={{ mt: 1 }}>Send</Button>
</Box>
```

**Mobile (`mobile/src/`)**

* Update `package.json`:

```json
"dependencies": {
  // ... existing ...
  "@react-native-voice/voice": "^3.2.4"
}
```

* Update `Practice.js`:

```javascript
import Voice from '@react-native-voice/voice';
// ...
const [isRecording, setIsRecording] = useState(false);

useEffect(() => {
  Voice.onSpeechResults = (e) => {
    const transcript = e.value[0];
    setChatInput(transcript);
    sendChat(transcript);  // Auto-send voice input
  };
  Voice.onSpeechEnd = () => setIsRecording(false);
  // Existing setup...
}, [navigation, mode]);

const toggleRecording = async () => {
  if (isRecording) {
    await Voice.stop();
  } else {
    await Voice.start('en-US');
    setIsRecording(true);
  }
};

const sendChat = (text = chatInput) => {
  if (wsRef.current && text) {
    wsRef.current.send(text);
    setChatMessages((prev) => [...prev, { sender: 'You', text }]);
    setChatInput('');
  }
};
// ...
<View style={{ marginVertical: 10 }}>
  <FlatList
    data={chatMessages}
    keyExtractor={(_, idx) => idx.toString()}
    renderItem={({ item }) => (
      <Text style={{ fontSize: 16 }}>{`${item.sender}: ${item.text}`}</Text>
    )}
    style={{ maxHeight: 200 }}
  />
  <View style={{ flexDirection: 'row', alignItems: 'center' }}>
    <TextInput
      placeholder="Ask the AI Tutor or use voice"
      value={chatInput}
      onChangeText={setChatInput}
      style={{ flex: 1, borderBottomWidth: 1, marginVertical: 10 }}
      onSubmitEditing={() => sendChat()}
    />
    <Button title={isRecording ? 'Stop' : 'Mic'} onPress={toggleRecording} />
  </View>
  <Button title="Send" onPress={() => sendChat()} />
</View>
```

*   **Permissions** (Android):

    * Add to `mobile/android/app/src/main/AndroidManifest.xml`:

    ```xml
    <uses-permission android:name="android.permission.RECORD_AUDIO" />
    ```

***

#### 2. Make It Conversational

* **Goal**: Enable multi-turn conversations with context retention (e.g., “What’s the next step?” after an explanation).
* **Implementation**: Add session context to WebSocket, improve AI responses.

**Backend**

* Update `ai_tutor.py`:

```python
from fastapi import WebSocket
from typing import Dict
import json

# In-memory context store (replace with Redis for production)
chat_sessions: Dict[str, list] = {}

@router.websocket("/chat/{user_id}")
async def chat_websocket(websocket: WebSocket, user_id: str, db: Session = Depends(get_db)):
    await websocket.accept()
    if user_id not in chat_sessions:
        chat_sessions[user_id] = []
    
    while True:
        message = await websocket.receive_text()
        # Append user message to context
        chat_sessions[user_id].append({"role": "user", "content": message})
        
        # Build prompt with context (last 5 messages for brevity)
        context = "\n".join([f"{m['role']}: {m['content']}" for m in chat_sessions[user_id][-5:]])
        prompt = f"Act as an SAT tutor. Respond conversationally with hints, explanations, or Q&A based on context:\n{context}"
        response = ai_respond(prompt)
        
        # Append AI response to context
        chat_sessions[user_id].append({"role": "assistant", "content": response})
        await websocket.send_text(response)
```

**Web**

* Update `practice.js` (already conversational via WebSocket):
  * No additional changes needed—context handled backend-side.
  * Example flow: “How do I solve x^2 + 5x + 6 = 0?” → “Factor it.” → “What’s the next step?” → “Set each factor to zero.”

**Mobile**

* Update `Practice.js`:
  * Same as web—WebSocket handles context.
  * Add a welcome message for better UX:

```javascript
useEffect(() => {
  // ... existing ...
  wsRef.current.onopen = () => {
    wsRef.current.send("Hi, I’m your SAT tutor. How can I help?");
    setChatMessages([{ sender: 'AI', text: "Hi, I’m your SAT tutor. How can I help?" }]);
  };
}, [navigation, mode]);
```

***

### Updated Files

* **Backend**: `routes/ai_tutor.py`.
* **Web**: `practice.js`.
* **Mobile**: `package.json`, `Practice.js`, Android permissions.

### Testing Plan

#### Setup

* Install mobile dependency: `cd mobile && npm install @react-native-voice/voice`.
* Rebuild mobile: `npx react-native run-android`.
* Start backend: `cd api && uvicorn main:app --reload`.
* Web: `cd web && npm run dev`.

#### Test Cases

1. **Voice Input**:
   * Web: Click mic, say “Explain this question” → Text appears, AI responds.
   * Mobile: Tap mic, say “How do I solve this?” → Same behavior.
2. **Conversational Flow**:
   * Web/Mobile: Ask “How do I factor x^2 + 5x + 6?” → AI explains.
   * Follow up: “What’s next?” → AI continues (e.g., “Set (x+2)(x+3) = 0”).
   * Ask “Give me a hint” → AI provides a nudge (e.g., “Think about factors of 6”).
3. **Context Retention**:
   * Ask “What’s the quadratic formula?” then “How do I use it?” → AI links responses.

#### Expected Results

* Voice input transcribes accurately, triggers chat.
* AI maintains context across turns (e.g., remembers previous question).
* No WebSocket disconnects or errors.

***

### Notes

* **Voice Quality**: Depends on device/browser mic; test in noisy environments.
* **Context**: In-memory `chat_sessions` is temporary—use Redis for persistence.
* **AI Model**: Assumes fine-tuned LLM; tweak prompts if using GPT-4o.

### Conclusion

* **Voice Input**: Added to web (Web Speech API) and mobile (`react-native-voice`).
* **Conversational**: Backend retains context, enabling multi-turn dialogue.

The AI tutor now feels like a real conversation partner with voice support. Test it—let me know if you want more (e.g., voice output) or adjustments!
