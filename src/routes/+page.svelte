<script lang="ts">
    let tituloDoJogo: string = "Campo-Minado";
    import Rain from '$lib/Rain.svelte';
</script>

<main class="home-container">
    <Rain amount={50} />

    <h1 class="title">Campo Minado</h1>

<a class="menu" href="/jogar">Iniciar Jogo</a>
    <a class="menu" href="/sobre">Sobre</a>
</main>