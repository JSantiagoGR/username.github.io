# username.github.io

<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, viewport-fit=cover">
    <title>Flashcard AI Studio Universal</title>
    
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="default">
    <meta name="apple-mobile-web-app-title" content="Flashcards IA">

    <script src="https://unpkg.com/lucide@latest"></script>
    <style>
        :root {
            --bg-color: #f6f8fa;
            --card-bg: #ffffff;
            --text-main: #1f2328;
            --text-muted: #657180;
            --accent: #007aff; 
            --border: #e1e4e8;
        }

        @media (prefers-color-scheme: dark) {
            :root {
                --bg-color: #161b22;
                --card-bg: #0d1117;
                --text-main: #f0f6fc;
                --text-muted: #8b949e;
                --accent: #2f81f7;
                --border: #30363d;
            }
        }

        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
            -webkit-tap-highlight-color: transparent;
        }

        /* CORRECCIÓN: Permitir scroll natural en todo el dispositivo */
        body {
            background-color: var(--bg-color);
            color: var(--text-main);
            min-height: 100vh;
            padding-top: env(safe-area-inset-top); 
        }

        header {
            background: var(--card-bg);
            padding: 16px;
            border-bottom: 1px solid var(--border);
            display: flex;
            justify-content: space-between;
            align-items: center;
            position: sticky;
            top: 0;
            z-index: 10;
        }

        header h1 { font-size: 22px; font-weight: 700; }

        /* CORRECCIÓN: Ajuste de márgenes para que las tarjetas no se escondan detrás del panel inferior */
        main {
            padding: 16px;
            padding-bottom: 200px; 
        }

        .category-tabs {
            display: flex;
            gap: 8px;
            overflow-x: auto;
            padding-bottom: 12px;
            margin-bottom: 16px;
            scrollbar-width: none;
            position: sticky;
            top: 60px;
            z-index: 9;
            background: var(--bg-color);
        }
        .category-tabs::-webkit-scrollbar { display: none; }

        .tab {
            background: var(--card-bg);
            padding: 8px 16px;
            border-radius: 20px;
            font-size: 14px;
            font-weight: 500;
            border: 1px solid var(--border);
            white-space: nowrap;
            cursor: pointer;
            color: var(--text-main);
        }

        .tab.active {
            background: var(--accent);
            color: white;
            border-color: var(--accent);
        }

        .flashcards-grid {
            display: grid;
            grid-template-columns: 1fr;
            gap: 16px;
        }

        .flashcard-wrapper {
            perspective: 1000px;
            min-height: 180px;
            cursor: pointer;
        }

        .flashcard {
            width: 100%;
            height: 100%;
            position: relative;
            transform-style: preserve-3d;
            transition: transform 0.6s cubic-bezier(0.4, 0, 0.2, 1);
        }

        .flashcard.flipped { transform: rotateY(180deg); }

        .card-front, .card-back {
            position: absolute;
            width: 100%;
            height: 100%;
            backface-visibility: hidden;
            background: var(--card-bg);
            border-radius: 16px;
            padding: 20px;
            display: flex;
            flex-direction: column;
            border: 1px solid var(--border);
            box-shadow: 0 4px 12px rgba(0,0,0,0.05);
        }

        .card-back {
            transform: rotateY(180deg);
        }

        .card-header-area {
            display: flex;
            justify-content: space-between;
            align-items: flex-start;
            margin-bottom: 8px;
            width: 100%;
        }

        .card-tag {
            font-size: 11px;
            text-transform: uppercase;
            font-weight: 700;
            color: var(--accent);
            letter-spacing: 0.5px;
        }

        .card-title {
            font-size: 24px;
            font-weight: 700;
            margin-bottom: auto;
            color: var(--text-main);
        }

        .card-body {
            font-size: 15px;
            line-height: 1.6;
            color: var(--text-main);
            overflow-y: auto;
            white-space: pre-line;
        }

        .delete-btn {
            background: none;
            border: none;
            color: #ff3b30;
            cursor: pointer;
            padding: 2px;
        }

        .creation-panel {
            position: fixed;
            bottom: 0;
            left: 0;
            right: 0;
            background: var(--card-bg);
            border-top: 1px solid var(--border);
            padding: 16px 16px calc(16px + env(safe-area-inset-bottom)); 
            display: flex;
            flex-direction: column;
            gap: 10px;
            box-shadow: 0 -4px 16px rgba(0,0,0,0.08);
            z-index: 20;
        }

        .input-row {
            display: flex;
            gap: 8px;
        }

        input, select {
            background: var(--bg-color);
            border: 1px solid var(--border);
            padding: 14px;
            border-radius: 12px;
            font-size: 16px;
            outline: none;
            color: var(--text-main);
        }

        input { flex: 1; }
        select { width: 120px; }

        .submit-btn {
            background: var(--accent);
            color: white;
            border: none;
            padding: 14px;
            border-radius: 12px;
            font-weight: 600;
            font-size: 16px;
            display: flex;
            justify-content: center;
            align-items: center;
            gap: 6px;
            cursor: pointer;
        }

        .submit-btn:disabled { opacity: 0.6; }

        .api-setup-btn {
            background: none;
            border: none;
            color: var(--text-muted);
            font-size: 12px;
            text-decoration: underline;
            cursor: pointer;
            text-align: center;
            margin-top: 2px;
        }

        .empty-state {
            text-align: center;
            color: var(--text-muted);
            margin-top: 60px;
        }
    </style>
</head>
<body>

    <header>
        <h1>Mis Flashcards</h1>
        <button onclick="addCategory()" style="background:none; border:none; color:var(--accent); font-weight:600; font-size:16px;">+ Nueva Lista</button>
    </header>

    <main>
        <div class="category-tabs" id="categoryTabs"></div>
        <div class="flashcards-grid" id="flashcardsGrid"></div>
    </main>

    <div class="creation-panel">
        <div class="input-row">
            <input type="text" id="wordInput" placeholder="Escribe cualquier palabra..." autocomplete="off">
            <select id="categorySelect"></select>
        </div>
        <button class="submit-btn" id="generateBtn" onclick="generateFlashcard()">
            <i data-lucide="sparkles" size="18"></i> Consultar Inteligencia Artificial
        </button>
        <button class="api-setup-btn" onclick="setupApiKey()">Configurar Clave API Gemini</button>
    </div>

    <script>
        let state = JSON.parse(localStorage.getItem('flashcards_universal_state')) || {
            categories: ['Todos', 'Medicina', 'Alemán'],
            activeCategory: 'Todos',
            flashcards: []
        };

        let apiKey = localStorage.getItem('gemini_flashcard_key') || '';

        function saveState() {
            localStorage.setItem('flashcards_universal_state', JSON.stringify(state));
        }

        function setupApiKey() {
            const key = prompt("Pega aquí tu API Key de Google Gemini:", apiKey);
            if (key !== null) {
                apiKey = key.trim();
                localStorage.setItem('gemini_flashcard_key', apiKey);
                alert("Clave guardada con éxito.");
            }
        }

        async function askGeminiAI(word, category) {
            if (!apiKey) {
                alert("Por favor, configura tu API Key primero.");
                return null;
            }

            const promptTexto = `Eres un asistente de estudio bilingüe y experto en pedagogía. El usuario quiere aprender el término "${word}" dentro de la categoría "${category}".
Genera una respuesta EXCLUSIVAMENTE usando viñetas planas (utiliza el carácter "•") siguiendo estrictamente estas reglas de contexto:

Si la categoría es "Alemán":
1. Primero detecta si "${word}" está escrito en español o en alemán.
2. Si el término está en español: Tradúcelo al alemán.
3. Si el término está en alemán: Tradúcelo al español.
4. En AMBOS casos, debes extraer obligatoriamente los datos del término en alemán. Si el término no es un sustantivo (como un saludo o verbo), adáptalo coherentemente (por ejemplo, para saludos o verbos pon "Artículo: No aplica" o explica su uso).
Escribe la respuesta exactamente con esta estructura:
• Traducción: [La palabra traducida al idioma opuesto]
• Artículo: [DER, DIE o DAS en mayúsculas si es sustantivo, o "No aplica" si es verbo/saludo]
• Plural: [La forma plural en alemán si es sustantivo, o "No aplica"]
• Frase útil: [Una frase corta de ejemplo usando la palabra en alemán junto con su traducción al español entre paréntesis]

Si la categoría es "Medicina":
• Qué es: [Definición médica simplificada en español]
• Detalles anatómicos/Cantidad: [Datos anatómicos o numéricos si aplica]
• Función principal: [Función en el organismo]

Para cualquier otra categoría:
• Concepto central: [Explicación adaptada al tema]
• Información útil: [Dato clave para memorizar]

Sé conciso, directo y no agregues textos extras ni saludos. Solo las viñetas directas.`;

            const url = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=${apiKey}`;

            try {
                const response = await fetch(url, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({
                        contents: [{ parts: [{ text: promptTexto }] }]
                    })
                });

                const data = await response.json();
                
                if (data.error) {
                    alert(`Error de Google: ${data.error.message}`);
                    return null;
                }

                if (data.candidates && data.candidates[0].content.parts[0].text) {
                    return data.candidates[0].content.parts[0].text.trim();
                } else {
                    throw new Error("Respuesta inválida");
                }
            } catch (error) {
                console.error(error);
                alert("Error al conectar. Verifica tu clave API.");
                return null;
            }
        }

        function renderCategories() {
            const tabsContainer = document.getElementById('categoryTabs');
            const selectContainer = document.getElementById('categorySelect');
            
            tabsContainer.innerHTML = state.categories.map(cat => `
                <div class="tab ${state.activeCategory === cat ? 'active' : ''}" onclick="setCategory('${cat}')">
                    ${cat}
                </div>
            `).join('');

            selectContainer.innerHTML = state.categories.filter(cat => cat !== 'Todos').map(cat => `
                <option value="${cat}" ${state.activeCategory === cat ? 'selected' : ''}>${cat}</option>
            `).join('');
        }

        function setCategory(cat) {
            state.activeCategory = cat;
            renderFlashcards();
            renderCategories();
        }

        function addCategory() {
            const name = prompt("Nombre de la nueva lista:");
            if (name && !state.categories.includes(name)) {
                state.categories.push(name);
                saveState();
                renderCategories();
            }
        }

        function renderFlashcards() {
            const grid = document.getElementById('flashcardsGrid');
            const filtered = state.flashcards.filter(card => 
                state.activeCategory === 'Todos' || card.category === state.activeCategory
            );

            if (filtered.length === 0) {
                grid.innerHTML = `
                    <div class="empty-state">
                        <i data-lucide="sparkles" size="40"></i>
                        <p style="margin-top:10px;">No hay tarjetas aquí.<br>Escribe cualquier palabra abajo.</p>
                    </div>
                `;
                lucide.createIcons();
                return;
            }

            grid.innerHTML = filtered.map(card => `
                <div class="flashcard-wrapper" onclick="flipCard(this)">
                    <div class="flashcard">
                        <div class="card-front">
                            <div class="card-header-area">
                                <span class="card-tag">${card.category}</span>
                                <button class="delete-btn" onclick="deleteCard(event, ${card.id})">
                                    <i data-lucide="trash-2" size="16"></i>
                                </button>
                            </div>
                            <h2 class="card-title">${card.word}</h2>
                            <span style="font-size:11px; color:var(--text-muted); text-align:right; width:100%;">Tocar para ver información de la IA →</span>
                        </div>
                        <div class="card-back">
                            <div class="card-header-area">
                                <span class="card-tag" style="color:var(--text-muted)">Análisis de la IA</span>
                            </div>
                            <div class="card-body">${card.definition}</div>
                        </div>
                    </div>
                </div>
            `).join('');
            
            lucide.createIcons();
        }

        function flipCard(wrapper) {
            const card = wrapper.querySelector('.flashcard');
            card.classList.toggle('flipped');
        }

        function deleteCard(event, id) {
            event.stopPropagation(); 
            if(confirm("¿Eliminar esta tarjeta?")) {
                state.flashcards = state.flashcards.filter(c => c.id !== id);
                saveState();
                renderFlashcards();
            }
        }

        async function generateFlashcard() {
            const wordInput = document.getElementById('wordInput');
            const categorySelect = document.getElementById('categorySelect');
            const generateBtn = document.getElementById('generateBtn');
            
            const word = wordInput.value.trim();
            const category = categorySelect.value;

            if (!word) return;
            if (!apiKey) {
                setupApiKey();
                return;
            }

            generateBtn.disabled = true;
            generateBtn.innerHTML = `<i data-lucide="loader" class="animate-spin" size="18"></i> Pensando en tiempo real...`;
            lucide.createIcons();

            const aiDefinition = await askGeminiAI(word, category);

            if (aiDefinition) {
                state.flashcards.unshift({
                    id: Date.now(),
                    word,
                    definition: aiDefinition,
                    category
                });
                saveState();
                wordInput.value = '';
            }

            generateBtn.disabled = false;
            generateBtn.innerHTML = `<i data-lucide="sparkles" size="18"></i> Consultar Inteligencia Artificial`;
            
            renderFlashcards();
        }

        renderCategories();
        renderFlashcards();
    </script>
</body>
</html>
