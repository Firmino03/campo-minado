<script>
    // Cria números caindo aleatoriamente
    export let amount = 40;

    let items = Array.from({ length: amount }, () => ({
        left: Math.random() * 100,      // posição horizontal aleatória
        delay: Math.random() * 3,       // atraso aleatório na animação
        dur: 3 + Math.random() * 2,     // duração da animação
        num: Math.ceil(Math.random() * 5) // número 1-5
    }));
</script>

<div class="rain-container">
    {#each items as item}
        <span
            class="drop"
            style="
                left: {item.left}%;
                animation-delay: {item.delay}s;
                animation-duration: {item.dur}s;
            "
        >
            {item.num}
        </span>
    {/each}
</div>
