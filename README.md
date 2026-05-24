# username.github.io
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>Flashcard AI Studio Pro</title>
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

        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
            -webkit-tap-highlight-color: transparent;
        }

        body {
            background-color: var(--bg-color);
            color: var(--text-main);
            display: flex;
            flex-direction: column;
            height: 100vh;
            overflow: hidden;
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

        header h1 {
            font-size: 20px;
            font-weight: 700;
        }

        main {
            flex: 1;
            overflow-y: auto;
            padding: 16px;
            padding-bottom: 120px; 
        }

        .category-tabs {
            display: flex;
            gap: 8px;
            overflow-x: auto;
            padding-bottom: 12px;
            margin-bottom: 16px;
            scrollbar-width: none;
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

        .flashcard.flipped {
            transform: rotateY(180deg);
        }

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
            box-shadow: 0 4px 6px rgba(0,0,0,0.02);
        }

        .card-back {
            transform: rotateY(180deg);
            background: #fafafa;
            position: relative;
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
            font-size: 14px;
            line-height: 1.5;
            color: var(--text-main);
            overflow-y: auto;
            white-space: pre-line; /* Mantiene los saltos de línea para un orden limpio */
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
            padding: 12px 16px calc(12px + env(safe-area-inset-bottom)); 
            display: flex;
            flex-direction: column;
            gap: 8px;
            box-shadow: 0 -4px 12px rgba(0,0,0,0.05);
        }

        .input-row {
            display: flex;
            gap: 8px;
        }

        input, select {
            background: var(--bg-color);
            border: 1px solid var(--border);
            padding: 12px;
            border-radius: 10px;
            font-size: 15px;
            outline: none;
            color: var(--text-main);
        }

        input { flex: 1; }
        select { width: 120px; }

        .submit-btn {
            background: var(--accent);
            color: white;
            border: none;
            padding: 12px;
            border-radius: 10px;
            font-weight: 600;
            font-size: 15px;
            display: flex;
            justify-content: center;
            align-items: center;
            gap: 6px;
            cursor: pointer;
        }

        .submit-btn:disabled { opacity: 0.6; }

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
        <button onclick="addCategory()" style="background:none; border:none; color:var(--accent); font-weight:600; font-size:15px;">+ Nueva Lista</button>
    </header>

    <main>
        <div class="category-tabs" id="categoryTabs"></div>
        <div class="flashcards-grid" id="flashcardsGrid"></div>
    </main>

    <div class="creation-panel">
        <div class="input-row">
            <input type="text" id="wordInput" placeholder="Introduce un concepto o palabra..." autocomplete="off">
            <select id="categorySelect"></select>
        </div>
        <button class="submit-btn" id="generateBtn" onclick="generateFlashcard()">
            <i data-lucide="sparkles" size="18"></i> Generar con IA Especializada
        </button>
    </div>

    <script>
        // Inicialización de datos locales
        let state = JSON.parse(localStorage.getItem('flashcards_intelligent_state')) || {
            categories: ['Todos', 'Medicina', 'Alemán', 'Granja'],
            activeCategory: 'Todos',
            flashcards: [
                { id: 1, word: 'Músculo', definition: '• Qué es: Tejido blando y contráctil del cuerpo formado por fibras musculares.\n• Cantidad: El cuerpo humano tiene aproximadamente 650 músculos esqueléticos.\n• Función: Permite el movimiento, mantiene la postura y estabiliza las articulaciones.', category: 'Medicina' },
                { id: 2, word: 'Perro', definition: '• Traducción: der Hund\n• Artículo: DER (Masculino)\n• Plural: die Hunde\n• Frase útil: "Der Hund bellt im Garten" (El perro ladra en el jardín).', category: 'Alemán' }
            ]
        };

        // El motor cognitivo de la APP (Simula prompts especializados de IA)
        const generateIntelligentDefinition = (word, category) => {
            return new Promise((resolve) => {
                setTimeout(() => {
                    const cleanWord = word.trim().toLowerCase();
                    let response = "";

                    // Diccionario Dinámico de Inteligencia por categoría
                    if (category.toLowerCase() === 'alemán') {
                        // Base de datos inteligente para Alemán
                        const baseAleman = {
                            'perro': { art: 'DER', trad: 'Hund', plu: 'Hunde', ex: 'Der Hund ist treu. (El perro es fiel).' },
                            'gato': { art: 'DIE', trad: 'Katze', plu: 'Katzen', ex: 'Die Katze schläft. (El gato duerme).' },
                            'auto': { art: 'DAS', trad: 'Auto', plu: 'Autos', ex: 'Das Auto es schnell. (El coche es rápido).' },
                            'casa': { art: 'DIE', trad: 'Haus', plu: 'Häuser', ex: 'Ich bleibe zu Hause. (Me quedo en casa).' }
                        };

                        if (baseAleman[cleanWord]) {
                            const d = baseAleman[cleanWord];
                            response = `• Traducción: der/die/das ${d.trad}\n• Artículo: ${d.art}\n• Plural: die ${d.plu}\n• Ejemplo: "${d.ex}"`;
                        } else {
                            // Respuesta genérica inteligente si la palabra no está mapeada
                            response = `• Traducción aproximada: [${word.toUpperCase()}]\n• Artículo sugerido: Analizar si es masculino (der), femenino (die) o neutro (das).\n• Tip de estudio: Busca si termina en -ung, -heit, -keit (siempre son DIE).`;
                        }

                    } else if (category.toLowerCase() === 'medicina') {
                        // Base de datos inteligente para Medicina
                        const baseMedicina = {
                            'músculo': { def: 'Tejido compuesto por fibras contráctiles que generan movimiento.', cant: 'Aprox. 650 músculos esqueléticos en el cuerpo humano.', extra: 'Se dividen en tres tipos: esquelético, cardíaco y liso.' },
                            'hueso': { def: 'Órgano firme, duro y resistente que forma el endoesqueleto de los vertebrados.', cant: 'El cuerpo humano adulto tiene exactamente 206 huesos.', extra: 'El hueso más largo es el fémur y el más pequeño es el estribo.' },
                            'corazón': { def: 'Órgano muscular hueco que bombea sangre a todo el cuerpo.', cant: '1 órgano principal, dividido en 4 cavidades (2 aurículas, 2 ventrículos).', extra: 'Late unas 100,000 veces al día en promedio.' }
                        };

                        if (baseMedicina[cleanWord]) {
                            const m = baseMedicina[cleanWord];
                            response = `• Qué es: ${m.def}\n• Cantidad/Datos: ${m.cant}\n• Detalles clave: ${m.extra}`;
                        } else {
                            response = `• Definición Médica: Término clínico relacionado con "${word}".\n• Anatomía/Fisiología: Evaluar localización sistémica y función celular.\n• Nota clínica: Investigar patologías comunes asociadas a este concepto.`;
                        }

                    } else if (category.toLowerCase() === 'granja') {
                        const baseGranja = {
                            'gato': { def: 'Felino doméstico usado habitualmente para el control de plagas de roedores en graneros.', rol: 'Control biológico / Mascota.', dato: 'Tienen un excelente oído y visión nocturna ideales para la caza.' },
                            'vaca': { def: 'Mamífero rumiante grande criado para la producción de leche y carne.', rol: 'Ganadería principal.', dato: 'Una vaca promedio produce alrededor de 25-30 litros de leche al día.' }
                        };

                        if (baseGranja[cleanWord]) {
                            const g = baseGranja[cleanWord];
                            response = `• Animal: ${word}\n• Rol en la Granja: ${g.rol}\n• Qué es: ${g.def}\n• Dato curioso: ${g.dato}`;
                        } else {
                            response = `• Categoría Granja: Información sobre "${word}".\n• Relación rural: Especie, crianza o herramienta útil para el sector agrícola.`;
                        }
                    } else {
                        // Cualquier otra lista personalizada que cree el usuario
                        response = `• Concepto: ${word}\n• Categoría: ${category}\n• Breve explicación: Información sintética útil diseñada automáticamente para el repaso de ${category}.`;
                    }

                    resolve(response);
                }, 800);
            });
        };

        function saveState() {
            localStorage.setItem('flashcards_intelligent_state', JSON.stringify(state));
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
            const name = prompt("Nombre de la nueva lista de estudio:");
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
                        <i data-lucide="layers" size="40"></i>
                        <p style="margin-top:10px;">Lista vacía.<br>Escribe una palabra abajo para crear una tarjeta inteligente.</p>
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
                            <span style="font-size:11px; color:var(--text-muted); text-align:right; width:100%;">Tocar para revelar info →</span>
                        </div>
                        <div class="card-back">
                            <div class="card-header-area">
                                <span class="card-tag" style="color:var(--text-muted)">Información Clave</span>
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

            generateBtn.disabled = true;
            generateBtn.innerHTML = `<i data-lucide="loader" class="animate-spin" size="18"></i> Estructurando datos...`;
            lucide.createIcons();

            // Aquí se ejecuta el filtro por categoría
            const intelligentDefinition = await generateIntelligentDefinition(word, category);

            state.flashcards.unshift({
                id: Date.now(),
                word,
                definition: intelligentDefinition,
                category
            });

            saveState();
            wordInput.value = '';
            generateBtn.disabled = false;
            generateBtn.innerHTML = `<i data-lucide="sparkles" size="18"></i> Generar con IA Especializada`;
            
            renderFlashcards();
        }

        renderCategories();
        renderFlashcards();
    </script>
</body>
</html>
