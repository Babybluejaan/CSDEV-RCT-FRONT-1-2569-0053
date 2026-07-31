<script>

    import adapter from '@sveltejs/adapter-vercel';

    /** @type {import('@sveltejs/kit').Config} */
    const config = {
        kit: {
            adapter: adapter()
        }
    };

    export default config;

    import Banana from '../assets/img/Subtract.png';
    import CursorDefault from '../assets/img/nomal.png';
    import CursorHover from '../assets/img/hover.png';

    let globalToastText = $state('');
    let globalTimer;

    const clipboard = (node, { char, onCopy }) => {
        let currentChar = char;

        const handleClick = async () => {
            try {
                await navigator.clipboard.writeText(currentChar);
                onCopy(currentChar);
            } catch (err) {
                console.error('Copy failed', err);
            }
        };

        node.addEventListener('click', handleClick);

        return {
            update(newProps) {
                currentChar = newProps.char;
            },
            destroy() {
                node.removeEventListener('click', handleClick);
            }
        };
    };

    let activeCopiedId = $state(null);
    let cardTimer;

    const handleCopySuccess = (formattedText, emojiId) => {
        activeCopiedId = emojiId;
        globalToastText = formattedText;

        clearTimeout(cardTimer);
        clearTimeout(globalTimer);

        cardTimer = setTimeout(() => {
            activeCopiedId = null;
        }, 1200);

        globalTimer = setTimeout(() => {
            globalToastText = '';
        }, 1200);
    };

    const scrollToEmoji = () => {
        document.getElementById('body-explore')?.scrollIntoView({
            behavior: 'smooth',
            block: 'start'
        });
    };

    const truncateMiddle = (str, maxLength = 8) => {
        if (!str || str.length <= maxLength) return str;
        
        const charsToShow = Math.floor((maxLength - 3) / 2);
        const front = str.slice(0, charsToShow);
        const back = str.slice(-charsToShow);
        
        return `${front}...${back}`;
    };

    const fixedPositions = [
        { x: 12, y: 28, size: 70, rotate: -10, delay: 0, duration: 3.5 }, // ซ้ายบน
        { x: 8,  y: 55, size: 85, rotate: -15, delay: 0.5, duration: 4 },  // ซ้ายล่าง
        { x: 82, y: 25, size: 80, rotate: 20,  delay: 0.2, duration: 3 },  // ขวาบน
        { x: 81, y: 65, size: 70, rotate: 10,  delay: 0.8, duration: 3.8 } // ขวาล่าง
    ];

    const getEmojis = async () => {
        const res = await fetch('/api/emoji');
        if (!res.ok) throw new Error('Something went wrong while loading emojis.');

        const emojis = await res.json();

        floatingEmojis = [...emojis]
            .sort(() => Math.random() - 0.5)
            .slice(0, 4)
            .map((emoji, index) => ({
                ...emoji,
                ...fixedPositions[index]
            }));

        return emojis;
    };

    let emojisPromise = getEmojis();
    let floatingEmojis = $state([]);
</script>

<div 
    class="page-wrapper"
    style="
        --cursor-default: url('{CursorDefault}');
        --cursor-hover: url('{CursorHover}');
    "
>
    <header class="header">
        <div class="container">
            <div class="banana-logo">
                <img src={Banana} alt="Banana Logo" />
                <span class="banana-text">Banana</span>
            </div>
        </div>
    </header>

    <main class="main-body">
        <div class="app">
            <div class="hero">
                {#each floatingEmojis as emoji}
                    <img
                        class="floating-img"
                        src="/api/emoji/{emoji.id}"
                        alt={emoji.name}
                        style="
                            left: {emoji.x}%;
                            top: {emoji.y}%;
                            --rotate: {emoji.rotate}deg;
                            animation-delay: {emoji.delay}s;
                            animation-duration: {emoji.duration}s;
                        "
                    />
                {/each}

                <div class="body-content">
                    <h1 class="title">Take Banana</h1>
                    <p class="description">Copied png to use as your emoji</p>
                </div>

                <div class="button-container">
                    <button class="button-bar" onclick={scrollToEmoji}>
                        Explore banana
                    </button>
                </div>
            </div>

            <div id="body-explore" class="body-explore">Explore</div>

            <div class="frame-outer">
                <div class="emoji-container">
                    <div class="emoji-grid">
                        {#await emojisPromise}
                            <p class="status-msg">Loading emojis...</p>
                        {:then emojis}
                            {#each emojis as emoji (emoji.id)}
                                <button
                                    class:copied={activeCopiedId === emoji.id}
                                    class="emoji-card" 
                                    use:clipboard={{ 
                                        char: `:${emoji.name}:`, 
                                        onCopy: (text) => handleCopySuccess(text, emoji.id) 
                                    }}
                                >
                                    <img src="/api/emoji/{emoji.id}" class="emoji-img" alt={emoji.name} />
                                    {#if emoji.char}
                                        <span class="emoji-icon">{emoji.char}</span>
                                    {/if}
                                    <span class="emoji-text">:{truncateMiddle(emoji.name, 8)}:</span>
                                    
                                    {#if activeCopiedId === emoji.id}
                                        <span class="copy-popup">
                                            <span class="copied-text">Copied!</span>
                                        </span>
                                    {/if}
                                </button>
                            {/each}
                        {:catch error}
                            <p class="status-msg error">Error: {error.message}</p>
                        {/await}
                    </div>
                </div>
            </div>
        </div>

        {#if globalToastText}
            <p class="toast">Copied {globalToastText} to clipboard! ✨</p>
        {/if}
    </main>

    <footer class="footer">
        <div class="footer-content">
            <p class="footer-text">Ⓒ Copyright 2026 CSDEV KMITL. All rights reserved.</p>
        </div>
    </footer>
</div>

<style>
    @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap');

    :global(body) {
        background-color: #181818;
        margin: 0;
        font-family: 'Inter', sans-serif;
    }

    /* Page Layout */
    .page-wrapper {
        cursor: var(--cursor-default), auto;
        min-height: 100vh;
        display: flex;
        flex-direction: column;
    }

    /* Header */
    .header {
        width: 100%;
        max-width: 1024px;
        height: 72px;
        margin: 0 auto;
        padding: 14px 18px;
        cursor: var(--cursor-default), auto;
    }

    .container {
        padding-top: 14px;
    }

    .banana-logo {
        display: inline-flex;
        align-items: center;
        gap: 10px;
        cursor: var(--cursor-hover) 8 8, pointer;
    }

    .banana-logo img {
        width: 36px;
        height: 36px;
    }

    .banana-text {
        color: aliceblue;
        font-size: clamp(18px, 2vw, 24px);
        font-weight: 600;
        line-height: 100%;
    }

    /* Main Content */
    .main-body {
        color: #FFFFFF;
        flex: 1;
    }

    .body-content {
        text-align: center;
    }

    .title {
        font-weight: 600;
        font-size: clamp(40px, 8vw, 96px);
        line-height: 100%;
        margin: 20px 0 10px;
        padding: 100px 0 0;
    }

    .description {
        font-weight: 300;
        font-size: clamp(18px, 3vw, 32px);
        line-height: 100%;
        margin: 0;
        padding: 15px;
    }

    /* Button */
    .button-container {
        display: flex;
        justify-content: center;
        margin-top: clamp(8px, 2vw, 16px);
        position: relative;
        z-index: 2;
        padding: 25px;
    }

    .button-bar {
        background-color: #FF7A45;
        color: #000;
        border: none;
        width: clamp(180px, 25vw, 272px);
        height: clamp(42px, 5vw, 51px);
        padding: 0;
        border-radius: 8px;
        font-size: clamp(14px, 1.5vw, 16px);
        font-weight: 700;
        cursor: var(--cursor-hover) 8 8, pointer;
        transition: transform 0.2s ease;
    }

    .button-bar:hover {
        transform: scale(1.05);
    }

    .body-explore {
        font-size: clamp(18px, 2vw, 24px);
        font-weight: 600;
        text-align: center;
    }

    /* Hero Section & Floating Emojis */
    .hero {
        position: relative;
        max-width: 1200px;
        margin: 0 auto;
        min-height: 450px;
        padding: 40px 0 0;
        overflow: hidden;
    }

    .floating-img {
        position: absolute;
        width: clamp(45px, 7vw, 85px);
        height: auto;
        object-fit: contain;
        pointer-events: none;
        transform: rotate(var(--rotate));
        animation: float ease-in-out infinite;
    }

    @keyframes float {
        0%, 100% {
            margin-top: 0;
        }
        50% {
            margin-top: -20px;
        }
    }

    /* Emoji Frame */
    .frame-outer {
        width: 100%;
        max-width: 1024px;
        margin: 20px auto;
        padding: 12px;
        background: #1F1F1F;
        border-radius: 12px;
        box-sizing: border-box;
    }

    .emoji-grid {
        display: grid;
        grid-template-columns: repeat(9, 1fr);
        gap: 16px;
        justify-items: center;
    }

    .emoji-card {
        position: relative;
        width: 80px;
        height: 90px;
        background-color: transparent;
        color: white;
        border: none;
        border-radius: 8px;
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
        gap: 6px;
        padding: 8px;
        cursor: var(--cursor-default), auto;
        transition: background-color 0.2s ease;
        box-sizing: border-box;
        min-width: 0;
    }

    .emoji-card:hover {
        background-color: #333333;
        cursor: var(--cursor-hover) 8 8, pointer;
    }

    .emoji-img {
        width: 54px;
        height: 54px;
        object-fit: contain;
        border-radius: 2px;
    }

    .emoji-icon {
        font-size: 32px;
    }

    .emoji-text {
        color: #FFFFFF;
        font-size: 12px;
        font-weight: 400;
        text-align: center;
        width: 100%;
        white-space: nowrap;
    }

    .copy-popup {
        position: absolute;
        inset: 0;
        display: flex;
        justify-content: center;
        align-items: center;
        background: rgba(0, 0, 0, 0.75);
        color: white;
        z-index: 10;
        border-radius: 8px;
    }

    .copied-text {
        animation: copiedSlideDown 0.1s ease-out;
    }

    @keyframes copiedSlideDown {
        0% {
            opacity: 0;
            transform: translateY(-25px);
        }
        70% {
            opacity: 1;
            transform: translateY(5px);
        }
        100% {
            opacity: 1;
            transform: translateY(0);
        }
    }

    .status-msg {
        color: white;
        text-align: center;
        grid-column: 1 / -1;
    }

    .status-msg.error {
        color: #FF7A45;
    }

    .toast {
        color: #FF5A5A;
        text-align: center;
        font-weight: bold;
        height: 24px;
    }

    /* Footer */
    .footer {
        cursor: var(--cursor-default), auto;
    }

    .footer-content {
        background-color: #1D1D1D;
        padding: 16px;
    }

    .footer-text {
        color: #8D8D8D;
        margin: 0 auto;
        text-align: center;
        padding: 16px;
        font-size: clamp(12px, 1.5vw, 14px);
        cursor: var(--cursor-default), auto;
        user-select: none;
    }

    .footer-text:hover {
        cursor: var(--cursor-hover) 8 8, pointer;
    }

    /* Responsive Design */
    @media (max-width: 1100px) {
        .frame-outer {
            width: 90%;
        }

        .emoji-grid {
            grid-template-columns: repeat(5, 1fr);
        }
    }

    @media (max-width: 600px) {
        .emoji-grid {
            grid-template-columns: repeat(3, 1fr);
        }

        .title {
            font-size: 48px;
            padding-top: 60px;
        }

        .description {
            font-size: 20px;
        }

        .hero {
            min-height: 350px;
        }

        .floating-img {
            width: clamp(35px, 12vw, 60px);
        }

        .button-bar {
            border-radius: 6px;
        }
    }
</style>