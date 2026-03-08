<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>AI Website Builder Pro</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <style>
        /* CRITICAL: Mobile Viewport Fix */
        body { 
            min-height: 100dvh; 
            display: flex; 
            flex-direction: column; 
            overflow: hidden; 
            background-color: #0f172a;
            color: white;
            font-family: ui-sans-serif, system-ui, sans-serif;
        }
        .glass { background: rgba(30, 41, 59, 0.7); backdrop-filter: blur(12px); }
        .sidebar-scroll { max-height: calc(100dvh - 180px); overflow-y: auto; }
        #code-editor { font-family: 'Fira Code', monospace; resize: none; }
        .dot { width: 12px; height: 12px; border-radius: 50%; }
        /* Prevent textarea from breaking layout */
        footer { flex-shrink: 0; }
        main { flex-grow: 1; overflow: hidden; }
    </style>
</head>
<body>

    <div class="flex flex-1 overflow-hidden">
        <aside class="w-64 border-r border-slate-700 flex flex-col bg-slate-900 z-20">
            <div class="p-4 flex-shrink-0">
                <button onclick="createNewProject()" class="w-full bg-blue-600 hover:bg-blue-500 transition-colors py-2 px-4 rounded-lg font-bold flex items-center justify-center gap-2">
                    <i class="fas fa-plus text-sm"></i> New Project
                </button>
            </div>

            <div id="project-list" class="flex-1 sidebar-scroll px-3 space-y-1">
                </div>

            <div class="p-4 border-t border-slate-800 flex-shrink-0">
                <button onclick="toggleSettings(true)" class="flex items-center gap-3 text-slate-400 hover:text-white transition-colors w-full">
                    <i class="fas fa-cog"></i> <span>Settings</span>
                </button>
            </div>
        </aside>

        <main class="flex-1 flex flex-col relative">
            <header class="h-14 border-b border-slate-700 flex items-center justify-between px-4 bg-slate-900/50">
                <div class="flex gap-2">
                    <div class="dot bg-red-500"></div>
                    <div class="dot bg-yellow-500"></div>
                    <div class="dot bg-green-500"></div>
                </div>

                <div class="flex bg-slate-800 rounded-lg p-1">
                    <button id="btn-preview" onclick="setView('preview')" class="px-4 py-1 rounded-md text-sm font-medium transition-all bg-blue-600 text-white">Preview</button>
                    <button id="btn-code" onclick="setView('code')" class="px-4 py-1 rounded-md text-sm font-medium transition-all text-slate-400">Code</button>
                </div>

                <button onclick="downloadCode()" class="bg-slate-700 hover:bg-slate-600 px-3 py-1.5 rounded text-sm flex items-center gap-2">
                    <i class="fas fa-download"></i> <span class="hidden sm:inline">Download</span>
                </button>
            </header>

            <div class="flex-1 relative overflow-hidden bg-white">
                <iframe id="preview-frame" class="w-full h-full border-none"></iframe>
                <textarea id="code-editor" class="absolute inset-0 w-full h-full bg-slate-950 text-emerald-400 p-4 hidden" spellcheck="false"></textarea>
                
                <div id="loader" class="absolute inset-0 bg-slate-900/80 flex items-center justify-center hidden z-30">
                    <div class="flex flex-col items-center gap-4">
                        <div class="animate-spin rounded-full h-12 w-12 border-t-2 border-b-2 border-blue-500"></div>
                        <p class="text-blue-400 font-medium animate-pulse">Engineering Layout...</p>
                    </div>
                </div>
            </div>

            <footer class="p-4 bg-slate-900 border-t border-slate-700">
                <div class="max-w-4xl mx-auto flex gap-3">
                    <textarea id="user-prompt" 
                        class="flex-1 bg-slate-800 border border-slate-600 rounded-xl px-4 py-3 text-white focus:outline-none focus:ring-2 focus:ring-blue-500 resize-none h-[54px]"
                        placeholder="Describe the website you want to build..."></textarea>
                    <button onclick="generateWebpage()" id="send-btn" class="bg-blue-600 hover:bg-blue-700 text-white px-6 rounded-xl font-bold transition-all flex items-center justify-center">
                        <i class="fas fa-paper-plane"></i>
                    </button>
                </div>
            </footer>
        </main>
    </div>

    <div id="settings-modal" class="fixed inset-0 bg-black/60 backdrop-blur-sm z-50 flex items-center justify-center hidden">
        <div class="bg-slate-900 border border-slate-700 w-full max-w-md p-6 rounded-2xl shadow-2xl">
            <h2 class="text-xl font-bold mb-6 flex items-center gap-2">
                <i class="fas fa-sliders-h text-blue-500"></i> API Configuration
            </h2>
            
            <div class="space-y-4">
                <div>
                    <label class="block text-xs font-bold text-slate-500 uppercase mb-1">API Endpoint</label>
                    <input id="set-url" type="text" class="w-full bg-slate-800 border border-slate-700 rounded-lg px-3 py-2 text-sm" placeholder="https://openrouter.ai/api/v1/chat/completions">
                </div>
                <div>
                    <label class="block text-xs font-bold text-slate-500 uppercase mb-1">API Key</label>
                    <input id="set-key" type="password" class="w-full bg-slate-800 border border-slate-700 rounded-lg px-3 py-2 text-sm" placeholder="sk-or-...">
                </div>
                <div>
                    <label class="block text-xs font-bold text-slate-500 uppercase mb-1">Model ID</label>
                    <input id="set-model" type="text" class="w-full bg-slate-800 border border-slate-700 rounded-lg px-3 py-2 text-sm" placeholder="deepseek/deepseek-r1:free">
                </div>
            </div>

            <div class="mt-8 flex gap-3">
                <button onclick="saveSettings()" class="flex-1 bg-blue-600 hover:bg-blue-500 py-2 rounded-lg font-bold">Save Changes</button>
                <button onclick="toggleSettings(false)" class="flex-1 bg-slate-800 hover:bg-slate-700 py-2 rounded-lg font-bold">Cancel</button>
            </div>
        </div>
    </div>

    <script>
        // --- CONSTANTS & STATE ---
        const DEFAULT_URL = "https://openrouter.ai/api/v1/chat/completions";
        const DEFAULT_MODEL = "deepseek/deepseek-r1:free";
        const SYSTEM_PROMPT = "You are an expert web developer. Output ONLY raw, functional HTML code including Tailwind CSS via CDN. Do not include markdown blocks like ```html. Ensure the design is modern and responsive.";

        let state = {
            projects: JSON.parse(localStorage.getItem('builder_projects') || '[]'),
            currentId: localStorage.getItem('builder_current_id') || null,
            settings: JSON.parse(localStorage.getItem('builder_settings') || JSON.stringify({
                url: DEFAULT_URL,
                key: '',
                model: DEFAULT_MODEL
            }))
        };

        // --- CORE LOGIC ---

        function init() {
            renderProjectList();
            if (state.currentId) {
                loadProject(state.currentId);
            } else {
                createNewProject();
            }
            // Populate settings inputs
            document.getElementById('set-url').value = state.settings.url;
            document.getElementById('set-key').value = state.settings.key;
            document.getElementById('set-model').value = state.settings.model;
        }

        async function generateWebpage() {
            const promptInput = document.getElementById('user-prompt');
            const prompt = promptInput.value.trim();
            if (!prompt) return;

            if (!state.settings.key) {
                alert("Please enter an API Key in Settings first.");
                toggleSettings(true);
                return;
            }

            setLoading(true);
            try {
                const response = await fetch(state.settings.url, {
                    method: 'POST',
                    headers: {
                        'Authorization': `Bearer ${state.settings.key}`,
                        'Content-Type': 'application/json',
                        'HTTP-Referer': window.location.origin,
                        'X-Title': 'AI Website Builder'
                    },
                    body: JSON.stringify({
                        model: state.settings.model,
                        messages: [
                            { role: "system", content: SYSTEM_PROMPT },
                            { role: "user", content: prompt }
                        ]
                    })
                });

                // ANTI-CRASH LOGIC
                const text = await response.text();
                
                if (!response.ok) {
                    throw new Error(`API Error (${response.status}): ${text}`);
                }

                const data = JSON.parse(text);
                let html = data.choices[0].message.content;

                // Clean markdown artifacts if model ignored instructions
                html = html.replace(/```html/g, '').replace(/```/g, '').trim();

                updateCurrentProject(html);
                promptInput.value = '';
            } catch (err) {
                console.error(err);
                alert(err.message);
            } finally {
                setLoading(false);
            }
        }

        // --- UTILS ---

        function updateCurrentProject(code) {
            const project = state.projects.find(p => p.id === state.currentId);
            if (project) {
                project.code = code;
                project.updatedAt = Date.now();
                saveState();
                renderView(code);
            }
        }

        function createNewProject() {
            const id = 'proj_' + Date.now();
            const newProj = {
                id,
                name: "New Website " + (state.projects.length + 1),
                code: "<html><body class='flex items-center justify-center h-screen bg-slate-50'> <h1 class='text-4xl font-bold text-slate-800'>Empty Project</h1> </body></html>",
                updatedAt: Date.now()
            };
            state.projects.unshift(newProj);
            state.currentId = id;
            saveState();
            renderProjectList();
            renderView(newProj.code);
        }

        function loadProject(id) {
            const project = state.projects.find(p => p.id === id);
            if (project) {
                state.currentId = id;
                saveState();
                renderProjectList();
                renderView(project.code);
            }
        }

        function renderView(code) {
            document.getElementById('preview-frame').srcdoc = code;
            document.getElementById('code-editor').value = code;
        }

        function renderProjectList() {
            const container = document.getElementById('project-list');
            container.innerHTML = state.projects.map(p => `
                <div onclick="loadProject('${p.id}')" class="p-2.5 rounded-lg cursor-pointer transition-all ${state.currentId === p.id ? 'bg-blue-600/20 text-blue-400 border border-blue-500/30' : 'text-slate-400 hover:bg-slate-800'}">
                    <div class="flex items-center gap-2 overflow-hidden">
                        <i class="fas fa-file-code text-xs"></i>
                        <span class="truncate text-sm font-medium">${p.name}</span>
                    </div>
                </div>
            `).join('');
        }

        function setView(type) {
            const preview = document.getElementById('preview-frame');
            const code = document.getElementById('code-editor');
            const btnP = document.getElementById('btn-preview');
            const btnC = document.getElementById('btn-code');

            if (type === 'preview') {
                preview.classList.remove('hidden');
                code.classList.add('hidden');
                btnP.classList.add('bg-blue-600', 'text-white');
                btnP.classList.remove('text-slate-400');
                btnC.classList.add('text-slate-400');
                btnC.classList.remove('bg-blue-600', 'text-white');
                // Update code from editor back to preview in case of manual edits
                preview.srcdoc = code.value;
                updateCurrentProject(code.value);
            } else {
                preview.classList.add('hidden');
                code.classList.remove('hidden');
                btnC.classList.add('bg-blue-600', 'text-white');
                btnC.classList.remove('text-slate-400');
                btnP.classList.add('text-slate-400');
                btnP.classList.remove('bg-blue-600', 'text-white');
            }
        }

        function saveSettings() {
            state.settings = {
                url: document.getElementById('set-url').value || DEFAULT_URL,
                key: document.getElementById('set-key').value,
                model: document.getElementById('set-model').value || DEFAULT_MODEL
            };
            saveState();
            toggleSettings(false);
        }

        function toggleSettings(show) {
            document.getElementById('settings-modal').classList.toggle('hidden', !show);
        }

        function setLoading(isLoading) {
            document.getElementById('loader').classList.toggle('hidden', !isLoading);
            document.getElementById('send-btn').disabled = isLoading;
        }

        function saveState() {
            localStorage.setItem('builder_projects', JSON.stringify(state.projects));
            localStorage.setItem('builder_current_id', state.currentId);
            localStorage.setItem('builder_settings', JSON.stringify(state.settings));
        }

        function downloadCode() {
            const code = document.getElementById('code-editor').value;
            const blob = new Blob([code], { type: 'text/html' });
            const url = URL.createObjectURL(blob);
            const a = document.createElement('a');
            a.href = url;
            a.download = 'index.html';
            a.click();
            URL.revokeObjectURL(url);
        }

        // Handle Enter key for textarea
        document.getElementById('user-prompt').addEventListener('keydown', function(e) {
            if (e.key === 'Enter' && !e.shiftKey) {
                e.preventDefault();
                generateWebpage();
            }
        });

        init();
    </script>
</body>
</html>
