<script>
  import { PUBLIC_API_URL } from '$env/static/public';

  let copiedText = '';

  const copyEmoji = async (char) => {
    try {
      await navigator.clipboard.writeText(char);
      copiedText = char;
      // Clear the toast message after 2 seconds
      setTimeout(() => (copiedText = ''), 2000);
    } catch (err) {
      console.error('คัดลอกไม่สำเร็จ:', err);
    }
  };

  const getEmojis = async () => {
    const res = await fetch(`/api/emoji`);
    if (!res.ok) throw new Error('โหลดข้อมูลไม่สำเร็จ');
    return await res.json();
  };

  // Store the promise in a variable to use in the HTML block
  let emojisPromise = getEmojis();
</script>

<header class="Banana">
  <div class="container">
    <!-- Header content can go here -->
  </div>
</header>

<main class="main-content">
  <div class="hero-container">
  
    <span class="floating-item top-left">😎</span>
    <span class="floating-item bottom-right">😵</span>

    <div class="content">
      <h1 class="title">Take Banana</h1>
      <p class="description">Copied png to use as your emoji</p>
    </div>

    <div class="bottom-bar">
      <button class="emoji-button" on:click={() => copyEmoji('🍌')}>
        Explore banana
      </button>
    </div>

    <div>
      <h2 class="text2">Explore</h2>
    </div>

    {#if copiedText}
      <p class="toast">คัดลอก {copiedText} ลง Clipboard แล้ว! ✨</p>
    {/if}

    <div class="frame-outer">
      <div class="emoji-grid">
        <!-- Svelte's await block handles loading, success, and error states -->
        {#await emojisPromise}
          <p style="color: white; text-align: center;">Loading emojis...</p>
        {:then emojis}
          <!-- Assuming your API returns an array of objects like { char: '🍎', name: 'Apple' } -->
          {#each emojis as emoji}
            <!-- Use button instead of div for better accessibility on click elements -->
            <button class="emoji-card" on:click={() => copyEmoji(emoji.char)}>
            <img src="/api/emoji/{emoji.id}" class="emoji-img" alt={emoji.name} />
              <!-- If your API returns images instead, swap this with: <img src={emoji.image_url} class="emoji-img" alt={emoji.name} /> -->
              <span class="emoji-icon">{emoji.char}</span>
              <span class="emoji-text">{emoji.name}</span>
            </button>
          {/each}
        {:catch error}
          <p style="color: #FF7A45; text-align: center;">Error: {error.message}</p>
        {/await}
      </div>
    </div>
  </div>
</main>

<footer class="footer">
  <div class="footer-content">
    <p class="footer-text">Ⓒ Copyright 2026 CSDEV KMITL. All rights reserved.</p>
  </div>
</footer>

<style>
  @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap');

  :global(body) {
    background-color: #181818;
    margin: 0;
    font-family: 'Inter', sans-serif;
  }

  .hero-container {
    position: relative;
    max-width: 900px;
    margin: 40px auto;
    padding: 40px 20px;
    text-align: center;
  }

  .floating-item {
    position: absolute;
    font-size: 40px;      
    width: 55px;          
    height: 55px;
    object-fit: cover;
    border-radius: 12px;   
    animation: float 3s ease-in-out infinite;
  }

  .top-left {
    top: 10%;
    left: 10%;
    transform: rotate(-12deg);
  }

  .bottom-right {
    bottom: 80%;
    right: 8%;
    transform: rotate(10deg);
    animation-delay: 1.5s;
  }

  @keyframes float {
    0% { transform: translateY(0px) rotate(0deg); }
    50% { transform: translateY(-12px) rotate(4deg); }
    100% { transform: translateY(0px) rotate(0deg); }
  }

  .container {
    max-width: 1200px;
    margin: 0 auto;
    padding: 0 50px;
    box-sizing: border-box; 
  }

  .main-content {
    min-height: calc(100vh - 140px);
    padding-bottom: 40px;
  }

  .content {
    max-width: 800px;
    margin: 0 auto;
    padding: 20px;
    color: hsl(0, 0%, 100%);
    text-align: center;
  }

  .title {
    font-size: 96px; /* Fixed 'size' to 'font-size' */
    margin: 0;
  }

  .emoji-button {
    background-color: #FF7A45;
    color: hsl(0, 0%, 0%);
    border: none;
    padding: 10px 20px;
    border-radius: 8px;
    font-weight: bold;
    cursor: pointer;
    width: 100%;
    max-width: 272px;
    height: 51px;
    font-family: 'Inter', sans-serif;
    transition: transform 0.2s;
  }

  .emoji-button:hover {
    transform: scale(1.05);
  }

  .bottom-bar {
    display: flex;
    justify-content: center;
    margin-top: 10px;
  }

  .text2 {
    color: hsl(0, 0%, 100%);
    text-align: center;
    margin-top: 40px;
  }

  .frame-outer {
    max-width: 1024px;
    width: 90%;                        
    min-height: 400px;                  
    background-color: #1F1F1F;  
    border: 1px solid #404040;  
    border-radius: 16px;
    padding: 30px;
    margin: 20px auto;                  
    box-shadow: 0 8px 24px rgba(0, 0, 0, 0.5);
    box-sizing: border-box; 
    overflow-y: auto; /* Allows scrolling if there are many emojis */             
  }

  .emoji-grid {
    display: flex;
    justify-content: center;
    align-items: center;
    gap: 16px;
    flex-wrap: wrap;
  }

  .emoji-card {
    width: 100px;                       
    height: 100px;
    background-color: hsl(0, 0%, 20%);
    border: 1px solid hsl(0, 0%, 35%);
    border-radius: 12px;
    display: flex;
    flex-direction: column;             
    justify-content: center;
    align-items: center;
    gap: 6px;                           
    cursor: pointer;
    transition: all 0.2s ease;
    padding: 8px;
    font-family: 'Inter', sans-serif;
  }

  .emoji-card:hover {
    background-color: hsl(0, 67%, 56%);
    transform: translateY(-4px);
    border-color: transparent;
  }

  .emoji-card:hover .emoji-text {
    color: hsl(0, 0%, 0%);
  }

  .emoji-icon {
    font-size: 32px;
  }

  .emoji-text {
    color: hsl(0, 0%, 100%);
    font-size: 12px;
    font-weight: 500;
    text-align: center;
  }

  .toast {
    color: hsl(0, 67%, 56%);
    text-align: center;
    font-weight: bold;
    height: 24px;
  }

  .footer {
    background-color: hsl(0, 0%, 10%);
    color: #8D8D8D;
    padding: 20px;
    text-align: center;
  }

  .footer-text {
    margin: 0;
  }
  
  .emoji-img {
    width: 40px;          
    height: 40px;         
    object-fit: contain;
  }
</style>