<script lang="ts">
    import Rain from '$lib/Rain.svelte';
</script>

<main class="home-container">
    <Rain amount={50} />

    <h1>💣 Campo Minado 💣</h1>

    <a href="/jogar" class="link-reset">
        <button class="button">Iniciar Jogo</button>
    </a>

    <a href="/sobre" class="link-reset">
        <button class="button">Sobre</button>
    </a>
</main>
