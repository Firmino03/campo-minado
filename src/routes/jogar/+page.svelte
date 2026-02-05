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

    function criarTabuleiroVazio(totalLinhas, totalColunas) {
        return Array.from(
            { length: totalLinhas },
            () => Array(totalColunas).fill(0)
        );
    }

    function criarMapaRevelado(totalLinhas, totalColunas) {
        return Array.from(
            { length: totalLinhas },
            () => Array(totalColunas).fill(false)
        );
    }

    function posicionarMinas(tabuleiro, quantidadeMinas) {
        let minasColocadas = 0;

        while (minasColocadas < quantidadeMinas) {
            const linha = Math.floor(Math.random() * tabuleiro.length);
            const coluna = Math.floor(Math.random() * tabuleiro[0].length);

            if (tabuleiro[linha][coluna] !== 9) {
                tabuleiro[linha][coluna] = 9;
                minasColocadas++;
            }
        }
    }

    function calcularNumeros(tabuleiro) {
        for (let linha = 0; linha < tabuleiro.length; linha++) {
            for (let coluna = 0; coluna < tabuleiro[0].length; coluna++) {
                if (tabuleiro[linha][coluna] === 9) continue;

                let minasAdjacentes = 0;

                for (let deltaLinha = -1; deltaLinha <= 1; deltaLinha++) {
                    for (let deltaColuna = -1; deltaColuna <= 1; deltaColuna++) {
                        const novaLinha = linha + deltaLinha;
                        const novaColuna = coluna + deltaColuna;

                        if (
                            novaLinha >= 0 &&
                            novaColuna >= 0 &&
                            novaLinha < tabuleiro.length &&
                            novaColuna < tabuleiro[0].length &&
                            tabuleiro[novaLinha][novaColuna] === 9
                        ) {
                            minasAdjacentes++;
                        }
                    }
                }

                tabuleiro[linha][coluna] = minasAdjacentes;
            }
        }
    }

    function revelarCelula(linha, coluna) {
        if (fimDeJogo) return;
        if (jogo.celulasReveladas[linha][coluna]) return;

        jogo.celulasReveladas[linha][coluna] = true;

        if (jogo.tabuleiro[linha][coluna] === 9) {
            fimDeJogo = true;
            resultadoJogo = 'derrota';
            salvarRanking();
            return;
        }

        pontuacao++;

        if (jogo.tabuleiro[linha][coluna] === 0) {
            revelarVizinhas(linha, coluna);
        }

        verificarVitoria();
    }

    function revelarVizinhas(linha, coluna) {
        for (let deltaLinha = -1; deltaLinha <= 1; deltaLinha++) {
            for (let deltaColuna = -1; deltaColuna <= 1; deltaColuna++) {
                const novaLinha = linha + deltaLinha;
                const novaColuna = coluna + deltaColuna;

                if (
                    novaLinha >= 0 &&
                    novaColuna >= 0 &&
                    novaLinha < TOTAL_LINHAS &&
                    novaColuna < TOTAL_COLUNAS &&
                    !jogo.celulasReveladas[novaLinha][novaColuna] &&
                    jogo.tabuleiro[novaLinha][novaColuna] !== 9
                ) {
                    revelarCelula(novaLinha, novaColuna);
                }
            }
        }
    }

    function verificarVitoria() {
        for (let linha = 0; linha < TOTAL_LINHAS; linha++) {
            for (let coluna = 0; coluna < TOTAL_COLUNAS; coluna++) {
                if (
                    jogo.tabuleiro[linha][coluna] !== 9 &&
                    !jogo.celulasReveladas[linha][coluna]
                ) {
                    return;
                }
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
        localStorage.setItem(
            'ranking-campo-minado',
            JSON.stringify(ranking)
        );
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
        <div class="tabuleiro">
            <table>
                {#each jogo.tabuleiro as linhaTabuleiro, indiceLinha}
                    <tr>
                        {#each linhaTabuleiro as celula, indiceColuna}
                            <td
                                class="celula {jogo.celulasReveladas[indiceLinha][indiceColuna] ? 'revelada' : ''}"
                                on:click={() =>
                                    revelarCelula(indiceLinha, indiceColuna)
                                }
                            >
                                {#if jogo.celulasReveladas[indiceLinha][indiceColuna]}
                                    {celula === 9 ? '💣' : celula || ''}
                                {/if}
                            </td>
                        {/each}
                    </tr>
                {/each}
            </table>
        </div>
    {/if}
</div>

{#if fimDeJogo}
    <div class="overlay">
        <div class="card card-derrota">
            <h2>
                {resultadoJogo === 'derrota'
                    ? '💥 Você explodiu!'
                    : '🎉 Você venceu!'}
            </h2>

            <ul class="lista-ganhadores">
                {#each ranking as jogador, indice}
                    <li>
                        <strong>{indice + 1}º</strong> — {jogador.nome} ({jogador.pontos})
                    </li>
                {/each}
            </ul>

            <a class="button link-reset" href="/">
                Voltar ao Menu
            </a>
        </div>
    </div>
{/if}
