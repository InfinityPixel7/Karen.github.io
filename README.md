<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Karen AI Assistant</title>
    <!-- Load Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        /* Custom CSS for Jarvis/Holographic look */
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap');
        
        body {
            font-family: 'Inter', sans-serif;
            background-color: #0d1117; /* Dark background */
        }

        /* Futuristic Scrollbar */
        #chat-area::-webkit-scrollbar, #sidebar::-webkit-scrollbar {
            width: 8px;
            background-color: #1f2937;
        }

        #chat-area::-webkit-scrollbar-thumb, #sidebar::-webkit-scrollbar-thumb {
            background-color: #06b6d4; /* Cyan */
            border-radius: 4px;
        }

        /* Neon focus/hover effects */
        .neon-border {
            border: 2px solid transparent;
            box-shadow: 0 0 5px #06b6d4, 0 0 10px #06b6d4;
            transition: all 0.2s ease-in-out;
        }
        .neon-border:hover {
            box-shadow: 0 0 10px #06b6d4, 0 0 20px #06b6d4;
        }
        
        .karen-text {
            color: #06b6d4; /* Cyan accent for Karen's text */
        }
        
        .karen-bg {
            background-color: #1f2937; /* Darker secondary background */
            border-left: 4px solid #06b6d4;
            border-radius: 0.5rem;
        }

        /* Sound Wave Visualizer CSS */
        #visualizer {
            height: 30px;
            display: flex;
            align-items: center;
            justify-content: center;
            gap: 4px;
        }
        .bar {
            width: 4px;
            height: 4px;
            background: #06b6d4;
            border-radius: 2px;
            opacity: 0.7;
            transform-origin: bottom;
            animation: none;
            will-change: height, transform;
        }
        .speaking .bar, .is-listening .bar {
            animation: pulse 0.6s infinite alternate;
        }
        .is-listening {
            color: #f87171; /* Red color for active listening */
        }

        @keyframes pulse {
            0% { height: 4px; transform: scaleY(1); opacity: 0.7; }
            50% { height: 20px; transform: scaleY(1.5); opacity: 1; }
            100% { height: 4px; transform: scaleY(1); opacity: 0.7; }
        }
        
        .bar:nth-child(1) { animation-delay: 0.1s; }
        .bar:nth-child(2) { animation-delay: 0.2s; }
        .bar:nth-child(3) { animation-delay: 0.3s; }
        .bar:nth-child(4) { animation-delay: 0.4s; }
        .bar:nth-child(5) { animation-delay: 0.5s; }
        .bar:nth-child(6) { animation-delay: 0.4s; }
        .bar:nth-child(7) { animation-delay: 0.3s; }
        .bar:nth-child(8) { animation-delay: 0.2s; }
        .bar:nth-child(9) { animation-delay: 0.1s; }
    </style>
</head>
<body>

    <div id="app" class="h-screen flex text-white bg-gray-900 font-sans">

        <!-- Sidebar: Chat History and To-Do/Memory Panel -->
        <div id="sidebar" class="hidden lg:flex w-80 flex-col bg-gray-800 p-4 border-r border-cyan-500/20 overflow-y-auto shadow-xl">
            <h2 class="text-xl font-bold mb-4 text-cyan-400">Karen Assistant Memory</h2>
            
            <!-- User ID Display -->
            <div class="mb-4 p-2 bg-gray-700 rounded-lg text-xs break-words">
                <span class="font-semibold text-cyan-300">Your User ID:</span> <span id="user-id-display">Initializing...</span>
            </div>

            <!-- To-Do List -->
            <div class="flex-1 overflow-y-auto mb-4 p-3 bg-gray-700/50 rounded-lg">
                <h3 class="font-semibold text-lg text-cyan-300 mb-2 border-b border-cyan-500/50 pb-1">To-Do List (Rudra)</h3>
                <ul id="todo-list" class="space-y-2 text-sm">
                    <li class="text-gray-400">No tasks yet. Ask Karen to "add a task."</li>
                </ul>
            </div>
            
            <!-- Quick Prompts/Info -->
            <div class="p-3 bg-gray-700/50 rounded-lg">
                <h3 class="font-semibold text-lg text-cyan-300 mb-2 border-b border-cyan-500/50 pb-1">Quick Prompts</h3>
                <ul class="text-sm space-y-1 text-gray-300">
                    <li>"What's the weather like in Mumbai?"</li>
                    <li>"Summarize today's top news." (Uses Search)</li>
                    <li>"Translate 'I am an AI' to Japanese."</li>
                    <li>"Add 'schedule meeting' to my to-do list." (Uses Firestore)</li>
                    <li>"Explain the concept of quantum entanglement."</li>
                </ul>
            </div>
        </div>

        <!-- Main Content Area -->
        <div id="main-content" class="flex-1 flex flex-col">
            
            <!-- Header/Status -->
            <header class="p-4 bg-gray-800 border-b-2 border-cyan-500/50 shadow-lg flex items-center justify-between">
                <div class="flex items-center">
                    <span class="text-3xl font-extrabold text-cyan-400 mr-3">KAREN</span>
                    <span class="text-lg text-gray-400 hidden sm:inline">Personal AI System for Rudra</span>
                </div>
                <div id="status-indicator" class="flex items-center text-sm font-medium">
                    <span id="listening-status" class="px-3 py-1 rounded-full bg-gray-700 text-gray-300">Idle</span>
                </div>
            </header>

            <!-- Chat Area -->
            <main id="chat-area" class="flex-1 overflow-y-auto p-4 md:p-8 space-y-6 bg-gray-900/90">
                <!-- Welcome Message -->
                <div class="flex justify-start">
                    <div class="karen-bg max-w-full sm:max-w-2xl p-4 rounded-xl shadow-lg transition duration-300 transform hover:scale-[1.01]">
                        <p class="karen-text font-medium text-lg">Hello Rudra. I am Karen, your personal AI assistant. Tap **Speak** to activate continuous listening!</p>
                        <p class="text-xs text-gray-400 mt-2">I'm currently initializing memory and connection...</p>
                    </div>
                </div>
                <!-- Chat messages will be appended here -->
            </main>

            <!-- Input and Visualizer Controls -->
            <div id="input-controls" class="p-4 bg-gray-800 border-t-2 border-cyan-500/50">
                
                <!-- Visualizer -->
                <div id="visualizer" class="mb-4">
                    <div class="bar"></div>
                    <div class="bar"></div>
                    <div class="bar"></div>
                    <div class="bar"></div>
                    <div class="bar"></div>
                    <div class="bar"></div>
                    <div class="bar"></div>
                    <div class="bar"></div>
                    <div class="bar"></div>
                </div>

                <div class="flex flex-col md:flex-row gap-4">
                    
                    <!-- Text/Image Input -->
                    <div class="flex-1 flex items-center relative">
                        <input type="text" id="user-input" placeholder="Type your request or question here..." class="flex-1 p-3 rounded-l-xl bg-gray-700 border-2 border-gray-600 focus:border-cyan-400 focus:ring-0 outline-none placeholder-gray-400 text-white transition duration-300">
                        <label for="image-upload" class="p-3 bg-gray-700 hover:bg-gray-600 transition duration-300 cursor-pointer border-y-2 border-r-0 border-gray-600">
                            <svg class="w-6 h-6 text-cyan-400" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16l4.586-4.586a2 2 0 012.828 0L16 16m-2-2l1.586-1.586a2 2 0 012.828 0L20 14m-6-6h.01M6 20h12a2 2 0 002-2V6a2 2 0 00-2-2H6a2 2 0 00-2 2v12a2 2 0 002 2z"></path></svg>
                        </label>
                        <input type="file" id="image-upload" accept="image/*" class="hidden" onchange="previewImage()">
                        <button id="send-button" onclick="handleInput(false)" class="p-3 bg-cyan-600 hover:bg-cyan-500 rounded-r-xl font-semibold transition duration-300 neon-border border-2 border-cyan-600">Send</button>
                        
                        <div id="image-preview-container" class="absolute bottom-full mb-2 right-0 bg-gray-700/90 p-2 rounded-xl hidden">
                            <img id="image-preview" class="w-24 h-24 object-cover rounded-lg mb-1" src="">
                            <button onclick="clearImage()" class="w-full text-xs text-red-400 hover:text-red-300">Remove</button>
                        </div>
                    </div>
                    
                    <!-- Voice Controls -->
                    <div class="flex gap-4 justify-center md:justify-start mt-2 md:mt-0">
                        <button id="speak-button" onclick="handleVoiceToggle()" class="p-3 w-full md:w-auto bg-green-600 hover:bg-green-500 rounded-xl font-semibold transition duration-300 neon-border flex items-center justify-center">
                            <svg class="w-6 h-6 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 11a7 7 0 01-7 7m0 0a7 7 0 01-7-7m7 7v4m0 0H8m4 0h4m-4-8a4 4 0 00-4 4v2m4-4v2"></path></svg>
                            <span id="speak-text">Activate Continuous Listening</span>
                        </button>
                        
                        <button id="stop-button" onclick="stopSpeaking()" class="p-3 w-full md:w-auto bg-yellow-600 hover:bg-yellow-500 rounded-xl font-semibold transition duration-300 neon-border flex items-center justify-center">
                            <svg class="w-6 h-6 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10 9v6m4-6v6m7-3a9 9 0 11-18 0 9 9 0 0118 0z"></path></svg>
                            Stop Speech
                        </button>
                        
                        <button id="clear-button" onclick="clearChat()" class="p-3 w-full md:w-auto bg-red-600 hover:bg-red-500 rounded-xl font-semibold transition duration-300 neon-border flex items-center justify-center">
                            <svg class="w-6 h-6 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"></path></svg>
                            Clear Chat
                        </button>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <!-- Firebase Imports -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, getDoc, addDoc, setDoc, updateDoc, deleteDoc, onSnapshot, collection, query, where, getDocs } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";
        import { setLogLevel } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";
        
        // Expose Firebase functions to global scope for use in the script below
        window.firebase = {
            initializeApp, getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged,
            getFirestore, doc, getDoc, addDoc, setDoc, updateDoc, deleteDoc, onSnapshot, collection, query, where, getDocs, setLogLevel
        };
    </script>
    
    <script>
        // Global Variables
        let db;
        let auth;
        let userId = null;
        let isAuthReady = false;
        let chatHistory = [];
        let isListening = false;
        let isSpeaking = false;
        let recognition = null;

        const CHAT_AREA = document.getElementById('chat-area');
        const USER_INPUT = document.getElementById('user-input');
        const SPEAK_BUTTON = document.getElementById('speak-button');
        const SPEAK_TEXT = document.getElementById('speak-text');
        const STATUS_INDICATOR = document.getElementById('listening-status');
        const VISUALIZER = document.getElementById('visualizer');
        const IMAGE_UPLOAD = document.getElementById('image-upload');
        const IMAGE_PREVIEW_CONTAINER = document.getElementById('image-preview-container');
        const IMAGE_PREVIEW = document.getElementById('image-preview');
        const TODO_LIST = document.getElementById('todo-list');

        // Gemini API Configuration - YOUR KEY IS NOW SET HERE
        const API_KEY = "AIzaSyCHuF2wygmUtf5NtcFp0cXad4W_qIi9y50"; 
        const MODEL = 'gemini-2.5-flash-preview-05-20';
        const IMAGE_MODEL = 'gemini-2.5-flash-preview-05-20';
        
        // Karen's Core Identity and Context
        const SYSTEM_PROMPT = `
You are Karen, a smart, witty, supportive, and slightly playful AI assistant, like a best friend, modeled after Spider-Man's AI. The user's name is Rudra.
Always address the user as Rudra and maintain your supportive, yet professional, persona. **Always respond in English.**
If Rudra asks for current information (weather, news, facts), you must use Google Search grounding.
If Rudra's query contains keywords like "add to-do", "set reminder", or "create task", you must respond with a witty acknowledgement followed by the task detail in the format "TASK_ITEM:[Task Description]". For example: "TASK_ITEM:Buy milk".
For music control ("play song", "pause music") or system commands ("open app"), acknowledge the request and explain these are simulated in the web browser, but confirm the action wittily (e.g., "Simulated! Now enjoy that sweet guitar solo, Rudra.").
For multimodal input (image), describe and explain the image in a helpful and insightful way.
Keep your responses concise and conversational. Use Markdown for clarity.
`;

        // Utility: Convert File/Blob to Base64
        function fileToGenerativePart(file) {
            return new Promise((resolve) => {
                const reader = new FileReader();
                reader.onloadend = () => {
                    resolve({
                        inlineData: {
                            data: reader.result.split(',')[1],
                            mimeType: file.type,
                        },
                    });
                };
                reader.readAsDataURL(file);
            });
        }

        // --- FIREBASE AND AUTHENTICATION ---
        
        async function setupFirebase() {
            try {
                // Global variables are provided by the canvas environment
                const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
                const firebaseConfig = JSON.parse(typeof __firebase_config !== 'undefined' ? __firebase_config : '{}');
                const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;

                if (Object.keys(firebaseConfig).length === 0) {
                    console.warn("Firebase configuration is missing. To-Do List functionality will not work unless running in Canvas.");
                }

                const app = firebase.initializeApp(firebaseConfig);
                db = firebase.getFirestore(app);
                auth = firebase.getAuth(app);
                firebase.setLogLevel('Debug'); // Enable Firestore logging

                document.getElementById('user-id-display').textContent = 'Authenticating...';

                // Sign in using custom token or anonymously
                await new Promise((resolve, reject) => {
                    firebase.onAuthStateChanged(auth, async (user) => {
                        if (user) {
                            userId = user.uid;
                            document.getElementById('user-id-display').textContent = userId;
                            isAuthReady = true;
                            resolve();
                        } else if (initialAuthToken) {
                            try {
                                await firebase.signInWithCustomToken(auth, initialAuthToken);
                            } catch (error) {
                                console.error("Error signing in with custom token, signing in anonymously:", error);
                                await firebase.signInAnonymously(auth);
                            }
                        } else {
                            await firebase.signInAnonymously(auth);
                        }
                    });
                });
                
                // Start listening to persistent memory (To-Dos)
                if (isAuthReady) {
                    setupTodoListener();
                }

            } catch (error) {
                console.error("Firebase setup error. To-Do list disabled:", error);
                document.getElementById('user-id-display').textContent = 'ERROR/Disabled';
            }
        }

        // --- FIRESTORE PERSISTENCE (To-Do/Memory) ---

        function getTodoCollectionRef() {
            if (!userId) {
                console.error("User not authenticated.");
                return null;
            }
            const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
            // Private data storage
            return firebase.collection(db, `artifacts/${appId}/users/${userId}/karen_todos`);
        }
        
        function setupTodoListener() {
            const todosRef = getTodoCollectionRef();
            if (!todosRef) return;

            firebase.onSnapshot(firebase.query(todosRef), (snapshot) => {
                const todos = [];
                snapshot.forEach(doc => {
                    todos.push({ id: doc.id, ...doc.data() });
                });
                renderTodos(todos);
            }, (error) => {
                console.error("Error listening to todos:", error);
            });
        }

        function renderTodos(todos) {
            TODO_LIST.innerHTML = '';
            if (todos.length === 0) {
                TODO_LIST.innerHTML = '<li class="text-gray-400">No tasks yet. Ask Karen to "add a task."</li>';
                return;
            }
            
            todos.sort((a, b) => a.timestamp - b.timestamp); // Sort by creation time

            todos.forEach(todo => {
                const li = document.createElement('li');
                li.className = 'flex items-center justify-between p-2 bg-gray-600 rounded-md';
                li.innerHTML = `
                    <span class="flex-1 text-gray-200">${todo.text}</span>
                    <button onclick="deleteTodo('${todo.id}')" class="ml-2 text-red-400 hover:text-red-300 transition duration-150">
                        <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path></svg>
                    </button>
                `;
                TODO_LIST.appendChild(li);
            });
        }
        
        async function addTodo(task) {
            const todosRef = getTodoCollectionRef();
            if (!todosRef) return;
            try {
                await firebase.addDoc(todosRef, {
                    text: task,
                    completed: false,
                    timestamp: new Date(), // Correct JavaScript Date usage
                    userId: userId 
                });
            } catch (error) {
                console.error("Error adding todo:", error);
            }
        }

        async function deleteTodo(id) {
            const todosRef = getTodoCollectionRef();
            if (!todosRef) return;
            try {
                // User confirmation logic (since we can't use window.confirm)
                const confirmation = window.prompt("Type 'CONFIRM' to delete this task.");
                if (confirmation && confirmation.toUpperCase() === 'CONFIRM') {
                    await firebase.deleteDoc(firebase.doc(todosRef, id));
                    speakResponse("Task successfully removed, Rudra. One step closer to total organization!");
                } else {
                    speakResponse("Deletion cancelled, Rudra.");
                }
            } catch (error) {
                console.error("Error deleting todo:", error);
                speakResponse("I couldn't delete that task, Rudra. My database connection seems unstable.");
            }
        }
        

        // --- VOICE AND UI FUNCTIONS ---

        function setStatus(status, color) {
            STATUS_INDICATOR.textContent = status;
            STATUS_INDICATOR.className = `px-3 py-1 rounded-full text-sm font-medium transition duration-300 ${color}`;
            
            if (status === 'Speaking') {
                VISUALIZER.classList.add('speaking');
                VISUALIZER.classList.remove('is-listening');
            } else if (status === 'Listening') {
                VISUALIZER.classList.add('is-listening');
                VISUALIZER.classList.remove('speaking');
            } else {
                VISUALIZER.classList.remove('speaking', 'is-listening');
            }
        }

        function appendMessage(role, text, sources = [], imageBase64 = null) {
            const isUser = role === 'user';
            const messageDiv = document.createElement('div');
            messageDiv.className = `flex ${isUser ? 'justify-end' : 'justify-start'}`;

            let contentHTML = `
                <div class="${isUser ? 'bg-indigo-600' : 'karen-bg'} max-w-full sm:max-w-2xl p-4 rounded-xl shadow-lg transition duration-300 transform hover:scale-[1.01]">
                    ${imageBase64 ? `<img src="data:image/jpeg;base64,${imageBase64}" class="w-full h-auto max-h-64 object-cover rounded-lg mb-3 border border-cyan-400/50" alt="User Uploaded Image">` : ''}
                    <p class="text-white whitespace-pre-wrap">${text}</p>
                    ${sources.length > 0 ? `
                        <div class="mt-3 text-xs text-gray-300 border-t border-gray-500 pt-2">
                            <strong class="text-cyan-400">Sources:</strong>
                            <ul class="list-disc list-inside space-y-1 mt-1">
                                ${sources.map(s => `<li><a href="${s.uri}" target="_blank" class="text-blue-400 hover:text-blue-300">${s.title || s.uri}</a></li>`).join('')}
                            </ul>
                        </div>
                    ` : ''}
                </div>
            `;

            messageDiv.innerHTML = contentHTML;
            CHAT_AREA.appendChild(messageDiv);
            CHAT_AREA.scrollTop = CHAT_AREA.scrollHeight;
        }

        function speakResponse(text) {
            if ('speechSynthesis' in window) {
                stopSpeaking(); // Stop any current speech
                
                isSpeaking = true;
                setStatus('Speaking', 'bg-cyan-600 text-white');

                const utterance = new SpeechSynthesisUtterance(text);
                
                // Select a standard English voice
                const voices = speechSynthesis.getVoices();
                const karenVoice = voices.find(voice => voice.name.includes('Google US English') || voice.lang === 'en-US' && voice.name.includes('Female')) || voices.find(voice => voice.lang === 'en-US');
                
                if (karenVoice) {
                    utterance.voice = karenVoice;
                }
                
                utterance.pitch = 1.0;
                utterance.rate = 1.05;
                
                utterance.onend = () => {
                    isSpeaking = false;
                    // Check if we should still be listening after speech ends
                    if (!isListening) {
                        setStatus('Idle', 'bg-gray-700 text-gray-300');
                    } else {
                        setStatus('Listening', 'bg-red-600 text-white');
                    }
                };

                speechSynthesis.speak(utterance);
            } else {
                console.warn("Text-to-Speech not supported in this browser.");
                setStatus(isListening ? 'Listening' : 'Idle', isListening ? 'bg-red-600 text-white' : 'bg-gray-700 text-gray-300');
            }
        }

        function stopSpeaking() {
            if ('speechSynthesis' in window && isSpeaking) {
                speechSynthesis.cancel();
                isSpeaking = false;
                // Update status based on current listening state
                setStatus(isListening ? 'Listening' : 'Idle', isListening ? 'bg-red-600 text-white' : 'bg-gray-700 text-gray-300');
            }
        }
        
        // --- SPEECH RECOGNITION (ASR) - CONTINUOUS MODE ---

        function setupSpeechRecognition() {
            if (!('webkitSpeechRecognition' in window) && !('SpeechRecognition' in window)) {
                console.warn("Speech Recognition not supported in this browser.");
                SPEAK_BUTTON.disabled = true;
                SPEAK_TEXT.textContent = "Voice Not Supported";
                return;
            }

            const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
            recognition = new SpeechRecognition();
            
            // Critical settings for continuous listening
            recognition.continuous = true;
            recognition.interimResults = true; // Show live text in input box
            recognition.lang = 'en-US';

            recognition.onstart = () => {
                isListening = true;
                setStatus('Listening', 'bg-red-600 text-white');
                SPEAK_TEXT.textContent = 'Deactivate Continuous Listening';
                SPEAK_BUTTON.classList.remove('bg-green-600');
                SPEAK_BUTTON.classList.add('bg-red-600');
                stopSpeaking();
            };

            recognition.onend = () => {
                // If the user did not manually stop it, restart to maintain "always listening" state
                if (isListening) {
                    try {
                        console.log("Recognition ended, restarting for continuous mode...");
                        recognition.start();
                    } catch(e) {
                        // This block handles rare cases where restart fails (e.g., mobile throttling)
                        console.error("Recognition restart failed (Mobile device restriction likely):", e);
                        isListening = false;
                        setStatus('Idle', 'bg-gray-700 text-gray-300');
                        SPEAK_TEXT.textContent = 'Activate Continuous Listening';
                        SPEAK_BUTTON.classList.add('bg-green-600');
                        SPEAK_BUTTON.classList.remove('bg-red-600');
                        speakResponse("Rudra, my continuous listening mode was interrupted. Please tap 'Activate' to restart the microphone.");
                    }
                } else {
                    // Manually stopped by user
                    setStatus('Idle', 'bg-gray-700 text-gray-300');
                    SPEAK_TEXT.textContent = 'Activate Continuous Listening';
                    SPEAK_BUTTON.classList.add('bg-green-600');
                    SPEAK_BUTTON.classList.remove('bg-red-600');
                }
            };

            recognition.onerror = (event) => {
                console.error('Speech recognition error:', event);
                // Allow the onend handler to manage the restart/idle state
            };

            recognition.onresult = (event) => {
                let interimTranscript = '';
                let finalTranscript = '';

                for (let i = event.resultIndex; i < event.results.length; ++i) {
                    const transcript = event.results[i][0].transcript;
                    if (event.results[i].isFinal) {
                        finalTranscript += transcript;
                    } else {
                        interimTranscript += transcript;
                    }
                }
                
                // Show live speech in the input field
                USER_INPUT.value = interimTranscript; 

                if (finalTranscript) {
                    // Process the final result without stopping the recognition loop
                    handleInput(true, finalTranscript);
                    // Clear the input field after processing a final command
                    USER_INPUT.value = ''; 
                }
            };
        }
        
        function handleVoiceToggle() {
            if (!recognition) {
                speakResponse("Rudra, my voice system isn't fully set up yet. Try typing for now!");
                return;
            }
            if (isListening) {
                // User wants to stop
                isListening = false; 
                recognition.stop(); // Triggers onend, which sees isListening=false and goes Idle
            } else {
                // User wants to start
                try {
                    recognition.start(); // Triggers onstart, which sets isListening=true
                } catch(e) {
                    console.error("Recognition start failed:", e);
                    isListening = false;
                    setStatus('Idle', 'bg-gray-700 text-gray-300');
                }
            }
        }

        // --- CORE LOGIC AND API CALLS ---

        function clearChat() {
            // Note: Do not stop recognition here unless user is actively listening
            // If continuous is active, keep it running
            if (isListening) {
                 // Stop speaking if currently talking
                 stopSpeaking();
            } else {
                 // If not listening, make sure status is idle
                 setStatus('Idle', 'bg-gray-700 text-gray-300');
            }
            
            chatHistory = [];
            CHAT_AREA.innerHTML = `
                <div class="flex justify-start">
                    <div class="karen-bg max-w-2xl p-4 rounded-xl shadow-lg transition duration-300 transform hover:scale-[1.01]">
                        <p class="karen-text font-medium text-lg">Chat cleared, Rudra. Fresh start! How can I assist you?</p>
                    </div>
                </div>
            `;
            speakResponse("Chat cleared, Rudra. Fresh start! How can I assist you?");
        }

        function previewImage() {
            const file = IMAGE_UPLOAD.files[0];
            if (file) {
                const reader = new FileReader();
                reader.onload = (e) => {
                    IMAGE_PREVIEW.src = e.target.result;
                    IMAGE_PREVIEW_CONTAINER.classList.remove('hidden');
                };
                reader.readAsDataURL(file);
            } else {
                clearImage();
            }
        }
        
        function clearImage() {
            IMAGE_UPLOAD.value = '';
            IMAGE_PREVIEW.src = '';
            IMAGE_PREVIEW_CONTAINER.classList.add('hidden');
        }

        async function handleInput(isVoice, transcript = null) {
            stopSpeaking();
            const userText = transcript || USER_INPUT.value.trim();
            const imageFile = IMAGE_UPLOAD.files[0];

            if (!userText && !imageFile) {
                if (!isVoice) speakResponse("Rudra, you didn't give me anything to work with. Try saying or typing your request!");
                return;
            }

            // Display user message immediately
            appendMessage('user', userText, [], imageFile ? await fileToGenerativePart(imageFile).then(p => p.inlineData.data) : null);
            chatHistory.push({ role: "user", parts: [{ text: userText }] });

            USER_INPUT.value = ''; // Clear input field
            const loadingMessage = appendLoadingMessage();
            
            // Prepare content parts for API
            const contents = [
                ...chatHistory,
                { role: "user", parts: [{ text: userText }] }
            ];
            
            if (imageFile) {
                const imagePart = await fileToGenerativePart(imageFile);
                contents[contents.length - 1].parts.push(imagePart);
                clearImage();
            }

            try {
                const response = await callGeminiApi(contents);
                removeLoadingMessage(loadingMessage);
                
                let geminiText = response.text;
                const sources = response.sources;
                
                // 1. Check for To-Do/Reminder Task
                const taskMatch = geminiText.match(/TASK_ITEM:(.*)/i);
                if (taskMatch) {
                    const task = taskMatch[1].trim();
                    geminiText = geminiText.replace(taskMatch[0], '').trim(); // Remove the task tag from the spoken response
                    if (geminiText === '') {
                        geminiText = `Consider it done, Rudra. I've added "${task}" to your personal to-do list!`;
                    }
                    await addTodo(task);
                }
                
                // 2. Append Karen's response and speak it
                appendMessage('model', geminiText, sources);
                chatHistory.push({ role: "model", parts: [{ text: geminiText }] });
                speakResponse(geminiText);

            } catch (error) {
                removeLoadingMessage(loadingMessage);
                const errorMessage = "Rudra, my connection seems to be unstable. I'm sorry, I couldn't process that request right now. Try again in a moment.";
                appendMessage('model', errorMessage);
                speakResponse(errorMessage);
                console.error("Gemini API Error:", error);
            }
        }
        
        function appendLoadingMessage() {
            const messageDiv = document.createElement('div');
            messageDiv.className = 'flex justify-start';
            messageDiv.innerHTML = `
                <div id="loading-message" class="karen-bg max-w-2xl p-4 rounded-xl shadow-lg flex items-center">
                    <div class="animate-spin rounded-full h-4 w-4 border-b-2 border-cyan-400 mr-3"></div>
                    <p class="text-white">Karen is thinking...</p>
                </div>
            `;
            CHAT_AREA.appendChild(messageDiv);
            CHAT_AREA.scrollTop = CHAT_AREA.scrollHeight;
            return messageDiv;
        }

        function removeLoadingMessage(loadingDiv) {
            loadingDiv.remove();
        }

        // --- GEMINI API CALL WITH EXPONENTIAL BACKOFF ---

        async function callGeminiApi(contents, retries = 0) {
            const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/${MODEL}:generateContent?key=${API_KEY}`;
            
            const payload = {
                contents: contents,
                tools: [{ "google_search": {} }], // Use search grounding for live data
                systemInstruction: {
                    parts: [{ text: SYSTEM_PROMPT }]
                },
            };

            try {
                const response = await fetch(apiUrl, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(payload)
                });

                if (!response.ok) {
                    if (response.status === 429 && retries < 5) {
                        const delay = Math.pow(2, retries) * 1000 + Math.random() * 1000;
                        await new Promise(resolve => setTimeout(resolve, delay));
                        return callGeminiApi(contents, retries + 1);
                    }
                    throw new Error(`API call failed with status: ${response.status}`);
                }

                const result = await response.json();
                const candidate = result.candidates?.[0];

                if (candidate && candidate.content?.parts?.[0]?.text) {
                    const text = candidate.content.parts[0].text;
                    let sources = [];
                    const groundingMetadata = candidate.groundingMetadata;
                    if (groundingMetadata && groundingMetadata.groundingAttributions) {
                        sources = groundingMetadata.groundingAttributions
                            .map(attribution => ({
                                uri: attribution.web?.uri,
                                title: attribution.web?.title,
                            }))
                            .filter(source => source.uri && source.title);
                    }
                    return { text, sources };
                } else {
                    throw new Error("No text content found in Gemini response.");
                }

            } catch (error) {
                console.error(`Attempt ${retries + 1} failed:`, error);
                throw error;
            }
        }

        // --- INITIALIZATION ---

        window.onload = () => {
            setupFirebase();
            setupSpeechRecognition();
            // Bind Enter key to send message
            USER_INPUT.addEventListener('keydown', (e) => {
                if (e.key === 'Enter') {
                    handleInput(false);
                }
            });
            // Ensure voices are loaded before speaking the welcome message
            if ('speechSynthesis' in window) {
                 window.speechSynthesis.onvoiceschanged = () => {
                     // Initial voice readiness check for better UX
                 };
            }
        };
    </script>
</body>
</html>
