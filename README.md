
<html lang="fr" class="dark">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Folio — 3D Book Showcase</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Google Fonts -->
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Playfair+Display:ital,wght@0,400;0,600;0,700;0,900;1,400&family=Inter:wght@300;400;500;600;700&display=swap" rel="stylesheet">
    <!-- Lucide Icons -->
    <script src="https://unpkg.com/lucide@latest"></script>

    <script>
        tailwind.config = {
            darkMode: 'class',
            theme: {
                extend: {
                    fontFamily: {
                        serif: ['Playfair Display', 'serif'],
                        sans: ['Inter', 'sans-serif'],
                    }
                }
            }
        }
    </script>

    <style>
        :root {
            --book-w: 190px;
            --book-h: 285px;
            --book-d: 32px;
        }

        @media (min-width: 1024px) {
            :root {
                --book-w: 220px;
                --book-h: 330px;
                --book-d: 38px;
            }
        }

        body {
            background-color: #080808;
            color: #f4f4f5;
            font-family: 'Inter', sans-serif;
            overflow-x: hidden;
        }

        /* Ambient Lighting */
        .glow-bg {
            background: radial-gradient(circle at 50% 45%, rgba(255, 255, 255, 0.05) 0%, rgba(0, 0, 0, 0) 70%);
        }

        /* Main 3D Perspective Stage */
        .stage-3d {
            perspective: 1600px;
            perspective-origin: 50% 40%;
        }

        .shelf-3d {
            transform-style: preserve-3d;
            display: flex;
            align-items: flex-end;
            justify-content: center;
            padding-bottom: 30px;
        }

        /* Container per book in the fan layout */
        .book-item {
            position: relative;
            transform-style: preserve-3d;
            transition: transform 0.5s cubic-bezier(0.25, 1, 0.3, 1), margin 0.5s ease;
            cursor: pointer;
            /* Overlap cascade effect */
            margin-left: -85px;
            margin-right: -85px;
            flex-shrink: 0;
        }

        @media (max-width: 768px) {
            .book-item {
                margin-left: -100px;
                margin-right: -100px;
            }
        }

        /* 3D Book Volume Element */
        .book-3d {
            width: var(--book-w);
            height: var(--book-h);
            position: relative;
            transform-style: preserve-3d;
            /* Default Angled Fan Rotation */
            transform: rotateY(-38deg) rotateX(12deg) rotateZ(3deg);
            transition: transform 0.5s cubic-bezier(0.16, 1, 0.3, 1);
            will-change: transform;
        }

        /* Faces */
        .face {
            position: absolute;
            backface-visibility: hidden;
            -webkit-backface-visibility: hidden;
        }

        /* Front Cover */
        .face-front {
            width: 100%;
            height: 100%;
            transform: translateZ(calc(var(--book-d) / 2));
            border-radius: 2px 6px 6px 2px;
            overflow: hidden;
            box-shadow: inset 4px 0 8px rgba(255, 255, 255, 0.2), inset -2px 0 6px rgba(0, 0, 0, 0.6);
            background-size: cover;
            background-position: center;
        }

        /* Back Cover */
        .face-back {
            width: 100%;
            height: 100%;
            transform: translateZ(calc(var(--book-d) / -2)) rotateY(180deg);
            background: #111113;
            border-radius: 6px 2px 2px 6px;
            box-shadow: inset 0 0 12px rgba(0,0,0,0.9);
        }

        /* Spine (Left Side) */
        .face-spine {
            width: var(--book-d);
            height: 100%;
            left: 0;
            top: 0;
            transform: translateX(-50%) rotateY(-90deg);
            border-radius: 2px;
            display: flex;
            align-items: center;
            justify-content: center;
            box-shadow: inset 0 0 10px rgba(0,0,0,0.7);
            overflow: hidden;
        }

        .face-spine span {
            writing-mode: vertical-rl;
            transform: rotate(180deg);
            font-family: 'Playfair Display', serif;
            font-size: 11px;
            letter-spacing: 0.15em;
            color: rgba(255, 255, 255, 0.9);
            text-transform: uppercase;
            white-space: nowrap;
        }

        /* Right Paper Edges */
        .face-right {
            width: var(--book-d);
            height: calc(100% - 6px);
            right: 0;
            top: 3px;
            transform: translateX(50%) rotateY(90deg);
            background: repeating-linear-gradient(
                180deg,
                #f8f6f0 0px,
                #e8e4da 1px,
                #d4cebf 2px,
                #f8f6f0 3px
            );
            box-shadow: inset 0 0 8px rgba(0,0,0,0.3);
        }

        /* Top Paper Edges */
        .face-top {
            width: calc(100% - 4px);
            height: var(--book-d);
            left: 2px;
            top: 0;
            transform: translateY(-50%) rotateX(90deg);
            background: repeating-linear-gradient(
                90deg,
                #f8f6f0 0px,
                #e8e4da 1px,
                #d4cebf 2px,
                #f8f6f0 3px
            );
            box-shadow: inset 0 0 5px rgba(0,0,0,0.3);
        }

        /* Realistic Floor Shadow */
        .shadow-floor {
            position: absolute;
            bottom: -30px;
            left: 5%;
            width: 90%;
            height: 30px;
            background: radial-gradient(ellipse at center, rgba(0, 0, 0, 0.9) 0%, rgba(0, 0, 0, 0) 75%);
            transform: rotateX(80deg) translateZ(-25px);
            transition: all 0.5s cubic-bezier(0.16, 1, 0.3, 1);
            pointer-events: none;
            opacity: 0.65;
        }

        /* --- FLAWLESS HOVER EFFECT: ELEVATE + CENTER FOCUS --- */
        .book-item:hover {
            z-index: 120 !important;
            margin-left: -20px;
            margin-right: -20px;
        }

        .book-item:hover .book-3d {
            /* Smoothly brings the book upright, centered towards viewer, lifted high */
            transform: translateY(-70px) translateZ(160px) rotateY(-14deg) rotateX(4deg) rotateZ(0deg) scale(1.15);
        }

        .book-item:hover .shadow-floor {
            transform: rotateX(80deg) translateZ(-50px) scale(1.5);
            opacity: 0.3;
            filter: blur(10px);
        }

        /* Giant Backdrop Typography */
        .hero-title {
            font-size: clamp(8rem, 28vw, 30rem);
            line-height: 0.72;
            letter-spacing: -0.05em;
            background: linear-gradient(180deg, rgba(255, 255, 255, 0.95) 0%, rgba(255, 255, 255, 0.12) 65%, rgba(255, 255, 255, 0.01) 100%);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
        }
    </style>
</head>
<body class="min-h-screen flex flex-col justify-between relative selection:bg-white selection:text-black">

    <!-- Ambient Glow -->
    <div class="fixed inset-0 glow-bg pointer-events-none z-0"></div>

    <!-- Header Navigation -->
    <header class="relative z-50 w-full px-6 py-6 lg:px-12 flex items-center justify-between border-b border-white/5 backdrop-blur-md bg-black/50">
        <div class="flex items-center gap-3">
            <div class="w-8 h-8 bg-white text-black flex items-center justify-center font-serif font-black text-xl rounded-sm shadow-md">
                F
            </div>
            <div class="leading-none">
                <span class="font-serif font-bold text-lg tracking-wider block">Folio</span>
                <span class="text-[9px] text-zinc-400 uppercase tracking-widest block font-sans">independent press</span>
            </div>
        </div>

        <nav class="hidden md:flex items-center space-x-9 text-xs font-medium text-zinc-400 tracking-wider uppercase">
            <a href="#" class="hover:text-white transition-colors duration-200">Books</a>
            <a href="#" class="hover:text-white transition-colors duration-200">Authors</a>
            <a href="#" class="hover:text-white transition-colors duration-200">Journal</a>
            <a href="#" class="hover:text-white transition-colors duration-200">More</a>
        </nav>

        <div class="flex items-center gap-4">
            <button class="text-xs uppercase tracking-wider text-zinc-400 hover:text-white font-medium px-2 py-2 transition-colors">
                Sign in
            </button>
            <button class="bg-white text-black hover:bg-zinc-200 text-xs uppercase tracking-wider font-bold px-5 py-2.5 rounded-full shadow-lg transition-all transform hover:scale-105">
                Start reading
            </button>
        </div>
    </header>

    <!-- Main Content Stage -->
    <main class="relative z-10 flex-1 flex flex-col items-center justify-center pt-2 pb-10 overflow-hidden min-h-[82vh]">
        
        <!-- Background "Folio" Serif Title -->
        <div class="absolute inset-0 flex items-start justify-center -z-10 select-none pointer-events-none pt-2">
            <h1 class="hero-title font-serif font-normal">Folio</h1>
        </div>

        <!-- 3D Carousel Stage -->
        <div class="w-full max-w-7xl mx-auto stage-3d my-auto pt-20 pb-12">
            <div id="shelf-container" class="shelf-3d px-6">
                <!-- Books injected via JS -->
            </div>
        </div>

        <!-- Category Menu -->
        <div class="z-30 flex flex-wrap justify-center items-center gap-3 md:gap-7 text-xs text-zinc-400 font-medium border border-white/10 py-3.5 px-8 backdrop-blur-md bg-black/60 rounded-full max-w-5xl mx-auto shadow-2xl">
            <button onclick="filterCategory('all', this)" class="cat-btn text-white font-semibold">Bestsellers</button>
            <span class="text-zinc-700">•</span>
            <button onclick="filterCategory('fiction', this)" class="cat-btn hover:text-white transition-colors">Fiction</button>
            <span class="text-zinc-700">•</span>
            <button onclick="filterCategory('nonfiction', this)" class="cat-btn hover:text-white transition-colors">Nonfiction</button>
            <span class="text-zinc-700">•</span>
            <button onclick="filterCategory('poetry', this)" class="cat-btn hover:text-white transition-colors">Poetry</button>
            <span class="text-zinc-700">•</span>
            <button onclick="filterCategory('classics', this)" class="cat-btn hover:text-white transition-colors">Classics</button>
            <span class="text-zinc-700">•</span>
            <button onclick="filterCategory('voices', this)" class="cat-btn hover:text-white transition-colors">Independent Voices</button>
            <span class="text-zinc-700">•</span>
            <button onclick="filterCategory('cards', this)" class="cat-btn hover:text-white transition-colors">Gift Cards</button>
        </div>
    </main>

    <!-- Inspection Modal -->
    <div id="book-modal" class="fixed inset-0 z-50 flex items-center justify-center p-4 bg-black/85 backdrop-blur-md opacity-0 pointer-events-none transition-all duration-300">
        <div id="modal-card" class="bg-zinc-950 border border-zinc-800 rounded-2xl max-w-2xl w-full p-8 relative shadow-2xl flex flex-col md:flex-row gap-8 transform scale-95 transition-transform duration-300">
            <button onclick="closeModal()" class="absolute top-4 right-4 text-zinc-500 hover:text-white p-2">
                <i data-lucide="x" class="w-5 h-5"></i>
            </button>
            
            <div class="w-48 h-72 flex-shrink-0 mx-auto rounded-lg overflow-hidden shadow-2xl border border-zinc-800">
                <img id="modal-img" src="" class="w-full h-full object-cover" alt="Book Cover">
            </div>

            <div class="flex-1 flex flex-col justify-between">
                <div>
                    <span id="modal-category" class="text-[10px] font-bold tracking-widest uppercase text-amber-400">Category</span>
                    <h2 id="modal-title" class="text-2xl font-serif font-bold text-white mt-1">Book Title</h2>
                    <p id="modal-author" class="text-sm text-zinc-400 mt-1">by Author Name</p>
                    <p id="modal-desc" class="text-xs text-zinc-300 mt-4 leading-relaxed">Book description...</p>
                </div>

                <div class="mt-6 flex items-center gap-4">
                    <button class="flex-1 bg-white text-black font-semibold text-xs py-3 rounded-full hover:bg-zinc-200 transition-colors">
                        Read Excerpt
                    </button>
                    <button id="modal-price" class="bg-zinc-900 border border-zinc-800 text-white font-semibold text-xs px-5 py-3 rounded-full hover:border-zinc-700 transition-colors">
                        $24.00 — Hardcover
                    </button>
                </div>
            </div>
        </div>
    </div>

    <!-- Footer -->
    <footer class="relative z-30 border-t border-white/5 py-6 px-6 text-center text-[11px] text-zinc-600">
        <div class="max-w-7xl mx-auto flex flex-col sm:flex-row justify-between items-center gap-3">
            <p>© 2026 Folio Independent Press. Interactive 3D Showcase.</p>
            <div class="flex gap-6">
                <a href="#" class="hover:text-zinc-400 transition-colors">Catalog</a>
                <a href="#" class="hover:text-zinc-400 transition-colors">Submissions</a>
                <a href="#" class="hover:text-zinc-400 transition-colors">Press Inquiries</a>
            </div>
        </div>
    </footer>

    <!-- Logic & Book Data -->
    <script>
        const books = [
            {
                id: 1,
                title: "A Brighter Tomorrow",
                author: "Elena Rostova",
                category: "fiction",
                price: "$24.50",
                spineColor: "linear-gradient(180deg, #9a3412, #431407)",
                cover: "https://images.unsplash.com/photo-1544716278-ca5e3f4abd8c?auto=format&fit=crop&w=600&q=80",
                desc: "An evocative narrative exploring human resilience across vast landscapes. A masterpiece of modern prose."
            },
            {
                id: 2,
                title: "The Weight of Time",
                author: "Marcus Vance",
                category: "nonfiction",
                price: "$29.00",
                spineColor: "linear-gradient(180deg, #b91c1c, #450a0a)",
                cover: "https://images.unsplash.com/photo-1512820790803-83ca734da794?auto=format&fit=crop&w=600&q=80",
                desc: "A deep philosophical investigation into memory, historical record, and our perception of temporal flow."
            },
            {
                id: 3,
                title: "Echoes of Silence",
                author: "Sarah Jenkins",
                category: "poetry",
                price: "$19.99",
                spineColor: "linear-gradient(180deg, #1d4ed8, #172554)",
                cover: "https://images.unsplash.com/photo-1532012197267-da84d127e765?auto=format&fit=crop&w=600&q=80",
                desc: "Poetic reflections on solitude, nature's quiet beauty, and internal emotional landscapes."
            },
            {
                id: 4,
                title: "Silent Horizon",
                author: "Kaito Tanaka",
                category: "fiction",
                price: "$26.00",
                spineColor: "linear-gradient(180deg, #0e7490, #083344)",
                cover: "https://images.unsplash.com/photo-1543002588-bfa74002ed7e?auto=format&fit=crop&w=600&q=80",
                desc: "Interconnected stories spanning Tokyo, Paris, and New York during pivotal moments of change."
            },
            {
                id: 5,
                title: "Quiet Waters",
                author: "Clara Dubois",
                category: "classics",
                price: "$22.00",
                spineColor: "linear-gradient(180deg, #047857, #022c22)",
                cover: "https://images.unsplash.com/photo-1516979187457-637abb4f9353?auto=format&fit=crop&w=600&q=80",
                desc: "Re-issue of the 1924 literary classic featuring restored typography and original archival artwork."
            },
            {
                id: 6,
                title: "The Great Architecture",
                author: "David Miller",
                category: "nonfiction",
                price: "$34.00",
                spineColor: "linear-gradient(180deg, #3f3f46, #09090b)",
                cover: "https://images.unsplash.com/photo-1497633762265-9d179a990aa6?auto=format&fit=crop&w=600&q=80",
                desc: "A breathtaking analysis of structural form and how physical spaces shape human thought and culture."
            },
            {
                id: 7,
                title: "Light & Gravity",
                author: "Hannah Arendt",
                category: "voices",
                price: "$21.50",
                spineColor: "linear-gradient(180deg, #a16207, #422006)",
                cover: "https://images.unsplash.com/photo-1457369804613-52c61a468e7d?auto=format&fit=crop&w=600&q=80",
                desc: "Essays exploring the intersection of modern physics, ethics, and human consciousness."
            },
            {
                id: 8,
                title: "Wilderness Within",
                author: "Arthur Pendelton",
                category: "fiction",
                price: "$27.00",
                spineColor: "linear-gradient(180deg, #15803d, #052e16)",
                cover: "https://images.unsplash.com/photo-1476275466078-4007374efbbe?auto=format&fit=crop&w=600&q=80",
                desc: "An adventurous journey into untamed lands and the interior search for personal redemption."
            }
        ];

        function renderShelf(items = books) {
            const shelf = document.getElementById('shelf-container');
            shelf.innerHTML = '';

            const total = items.length;

            items.forEach((book, idx) => {
                const item = document.createElement('div');
                item.className = 'book-item';
                
                // Z-index calculation for perfect depth stacking
                const centerOffset = idx - Math.floor(total / 2);
                const zIndex = total - Math.abs(centerOffset);
                item.style.zIndex = zIndex;

                item.innerHTML = `
                    <div class="book-3d">
                        <!-- Front Face -->
                        <div class="face face-front" style="background-image: url('${book.cover}');">
                            <div class="absolute inset-0 bg-gradient-to-t from-black/85 via-black/20 to-transparent p-4 flex flex-col justify-end">
                                <span class="text-[9px] uppercase font-bold tracking-widest text-amber-300/90">${book.category}</span>
                                <h4 class="font-serif font-bold text-xs md:text-sm text-white leading-tight">${book.title}</h4>
                            </div>
                        </div>

                        <!-- Back Face -->
                        <div class="face face-back"></div>

                        <!-- Spine Face -->
                        <div class="face face-spine" style="background: ${book.spineColor};">
                            <span>${book.title} — ${book.author}</span>
                        </div>

                        <!-- Top Face -->
                        <div class="face face-top"></div>

                        <!-- Right Face -->
                        <div class="face face-right"></div>
                    </div>

                    <!-- Floor Shadow -->
                    <div class="shadow-floor"></div>
                `;

                item.addEventListener('click', () => openModal(book));
                shelf.appendChild(item);
            });
        }

        function filterCategory(cat, btnElem) {
            document.querySelectorAll('.cat-btn').forEach(btn => {
                btn.className = 'cat-btn text-zinc-400 hover:text-white transition-colors';
            });
            btnElem.className = 'cat-btn text-white font-semibold';

            if (cat === 'all') {
                renderShelf(books);
            } else {
                const filtered = books.filter(b => b.category === cat);
                renderShelf(filtered.length ? filtered : books);
            }
        }

        function openModal(book) {
            document.getElementById('modal-img').src = book.cover;
            document.getElementById('modal-title').innerText = book.title;
            document.getElementById('modal-author').innerText = `by ${book.author}`;
            document.getElementById('modal-category').innerText = book.category;
            document.getElementById('modal-desc').innerText = book.desc;
            document.getElementById('modal-price').innerText = `${book.price} — Hardcover`;

            const modal = document.getElementById('book-modal');
            const card = document.getElementById('modal-card');

            modal.classList.remove('opacity-0', 'pointer-events-none');
            card.classList.remove('scale-95');
            card.classList.add('scale-100');
        }

        function closeModal() {
            const modal = document.getElementById('book-modal');
            const card = document.getElementById('modal-card');

            card.classList.remove('scale-100');
            card.classList.add('scale-95');
            modal.classList.add('opacity-0', 'pointer-events-none');
        }

        window.onload = function() {
            renderShelf();
            if (window.lucide) lucide.createIcons();
        };
    </script>
</body>
</html>
