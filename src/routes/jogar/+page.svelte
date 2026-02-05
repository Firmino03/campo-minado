<script>
    import { onMount } from 'svelte';

    const TOTAL_LINHAS = 10;
    const TOTAL_COLUNAS = 10;
    const TOTAL_MINAS = 15;

    let nomeJogador = '';
    let jogoIniciado = false;

    let fimDeJogo = false;
    let resultadoJogo = null;

    let pontuacao = 0;
    let ranking = [];

    function criarJogo() {
        const tabuleiro = criarTabuleiroVazio(TOTAL_LINHAS, TOTAL_COLUNAS);
        posicionarMinas(tabuleiro, TOTAL_MINAS);
        calcularNumeros(tabuleiro);

        const celulasReveladas = criarMapaRevelado(TOTAL_LINHAS, TOTAL_COLUNAS);
        pontuacao = 0;

        return { tabuleiro, celulasReveladas };
    }

    function criarTabuleiroVazio(linhas, colunas) {
        return Array.from({ length: linhas }, () => Array(colunas).fill(0));
    }

    function criarMapaRevelado(linhas, colunas) {
        return Array.from({ length: linhas }, () => Array(colunas).fill(false));
    }

    function posicionarMinas(tabuleiro, quantidadeMinas) {
        let minasColocadas = 0;

        while (minasColocadas < quantidadeMinas) {
            const l = Math.floor(Math.random() * tabuleiro.length);
            const c = Math.floor(Math.random() * tabuleiro[0].length);

            if (tabuleiro[l][c] !== 9) {
                tabuleiro[l][c] = 9;
                minasColocadas++;
            }
        }
    }

    function calcularNumeros(tabuleiro) {
        for (let l = 0; l < tabuleiro.length; l++) {
            for (let c = 0; c < tabuleiro[0].length; c++) {
                if (tabuleiro[l][c] === 9) continue;

                let minas = 0;

                for (let dl = -1; dl <= 1; dl++) {
                    for (let dc = -1; dc <= 1; dc++) {
                        const nl = l + dl;
                        const nc = c + dc;

                        if (
                            nl >= 0 &&
                            nc >= 0 &&
                            nl < tabuleiro.length &&
                            nc < tabuleiro[0].length &&
                            tabuleiro[nl][nc] === 9
                        ) {
                            minas++;
                        }
                    }
                }

                tabuleiro[l][c] = minas;
            }
        }
    }

    function revelarCelula(l, c) {
        if (fimDeJogo) return;
        if (jogo.celulasReveladas[l][c]) return;

        jogo.celulasReveladas[l][c] = true;

        if (jogo.tabuleiro[l][c] === 9) {
            fimDeJogo = true;
            resultadoJogo = 'derrota';
            salvarRanking();
            return;
        }

        pontuacao++;

        if (jogo.tabuleiro[l][c] === 0) {
            revelarVizinhas(l, c);
        }

        verificarVitoria();
    }

    function revelarVizinhas(l, c) {
        for (let dl = -1; dl <= 1; dl++) {
            for (let dc = -1; dc <= 1; dc++) {
                const nl = l + dl;
                const nc = c + dc;

                if (
                    nl >= 0 &&
                    nc >= 0 &&
                    nl < TOTAL_LINHAS &&
                    nc < TOTAL_COLUNAS &&
                    !jogo.celulasReveladas[nl][nc] &&
                    jogo.tabuleiro[nl][nc] !== 9
                ) {
                    revelarCelula(nl, nc);
                }
            }
        }
    }

    function verificarVitoria() {
        for (let l = 0; l < TOTAL_LINHAS; l++) {
            for (let c = 0; c < TOTAL_COLUNAS; c++) {
                if (
                    jogo.tabuleiro[l][c] !== 9 &&
                    !jogo.celulasReveladas[l][c]
                ) return;
            }
        }

        fimDeJogo = true;
        resultadoJogo = 'vitoria';
        salvarRanking();
    }

    function salvarRanking() {
        ranking.push({ nome: nomeJogador, pontos: pontuacao });
        ranking.sort((a, b) => b.pontos - a.pontos);
        ranking = ranking.slice(0, 3);
        localStorage.setItem('ranking-campo-minado', JSON.stringify(ranking));
    }

    function iniciarJogo() {
        if (!nomeJogador.trim()) return;
        jogo = criarJogo();
        jogoIniciado = true;
    }

    onMount(() => {
        const dados = localStorage.getItem('ranking-campo-minado');
        if (dados) ranking = JSON.parse(dados);
    });

    let jogo = criarJogo();
</script>

<!-- =======================
     TELA DO JOGO
     ======================= -->
<div class="tela-jogo">
    <h1 class="titulo-jogo">💣 Tente não explodir 💣</h1>

    {#if !jogoIniciado}
        <!-- CARD INÍCIO -->
        <div class="card-inicio-jogo">
            <h2 class="titulo-fim">Digite seu nome</h2>

            <input
                class="input-nome"
                placeholder="Seu nome"
                bind:value={nomeJogador}
            />

            <button class="button" on:click={iniciarJogo}>
                Iniciar jogo
            </button>
        </div>

    {:else}
        <!-- TABULEIRO -->
        <div class="tabuleiro">
            <table>
                {#each jogo.tabuleiro as linhaTabuleiro, l}
                    <tr>
                        {#each linhaTabuleiro as celula, c}
                            <td
                                class="celula {jogo.celulasReveladas[l][c] ? 'revelada' : ''}"
                                on:click={() => revelarCelula(l, c)}
                            >
                                {#if jogo.celulasReveladas[l][c]}
                                    {celula === 9 ? "💣" : celula || ""}
                                {/if}
                            </td>
                        {/each}
                    </tr>
                {/each}
            </table>
        </div>
    {/if}
</div>

<!-- =======================
     OVERLAY FIM DE JOGO
     ======================= -->
{#if fimDeJogo}
    <div class="overlay">
        <!-- 👇 CARD COM AJUSTE FINO -->
        <div class="card card-derrota">
            <h2>
                {resultadoJogo === 'derrota'
                    ? '💥 Você explodiu!'
                    : '🎉 Você venceu!'}
            </h2>

            <!-- 👇 LISTA CENTRALIZADA -->
            <ul class="lista-ganhadores">
                {#each ranking as j, i}
                    <li>
                        <strong>{i + 1}º</strong> — {j.nome} ({j.pontos})
                    </li>
                {/each}
            </ul>

            <a class="button link-reset" href="/">
                Voltar ao Menu
            </a>
        </div>
    </div>
{/if}
