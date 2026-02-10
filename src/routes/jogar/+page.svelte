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

    class CampoMinado {
        criarJogo() {
            const tabuleiro = this.criarTabuleiroVazio(TOTAL_LINHAS, TOTAL_COLUNAS);
            this.posicionarMinas(tabuleiro, TOTAL_MINAS);
            this.calcularNumeros(tabuleiro);

            const celulasReveladas = this.criarMapaRevelado(TOTAL_LINHAS, TOTAL_COLUNAS);
            pontuacao = 0;
            fimDeJogo = false;
            resultadoJogo = null;

            return { tabuleiro, celulasReveladas };
        }

        criarTabuleiroVazio(totalLinhas, totalColunas) {
            const tabuleiro = [];

            for (let i = 0; i < totalLinhas; i++) {
                const linha = [];

                for (let j = 0; j < totalColunas; j++) {
                    linha.push(0);
                }

                tabuleiro.push(linha);
            }

            return tabuleiro;
        }

        criarMapaRevelado(totalLinhas, totalColunas) {
            const mapa = [];

            for (let i = 0; i < totalLinhas; i++) {
                const linha = [];

                for (let j = 0; j < totalColunas; j++) {
                    linha.push(false);
                }

                mapa.push(linha);
            }

            return mapa;
        }

        posicionarMinas(tabuleiro, quantidadeMinas) {
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

        calcularNumeros(tabuleiro) {
            for (let linha = 0; linha < tabuleiro.length; linha++) {
                for (let coluna = 0; coluna < tabuleiro[0].length; coluna++) {
                    if (tabuleiro[linha][coluna] === 9) continue;

                    let minasAdjacentes = 0;

                    for (let dl = -1; dl <= 1; dl++) {
                        for (let dc = -1; dc <= 1; dc++) {
                            const novaLinha = linha + dl;
                            const novaColuna = coluna + dc;

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

        revelarCelula(linha, coluna) {
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
                this.revelarVizinhas(linha, coluna);
            }

            this.verificarVitoria();
        }

        revelarVizinhas(linha, coluna) {
            for (let dl = -1; dl <= 1; dl++) {
                for (let dc = -1; dc <= 1; dc++) {
                    const novaLinha = linha + dl;
                    const novaColuna = coluna + dc;

                    if (
                        novaLinha >= 0 &&
                        novaColuna >= 0 &&
                        novaLinha < TOTAL_LINHAS &&
                        novaColuna < TOTAL_COLUNAS &&
                        !jogo.celulasReveladas[novaLinha][novaColuna] &&
                        jogo.tabuleiro[novaLinha][novaColuna] !== 9
                    ) {
                        this.revelarCelula(novaLinha, novaColuna);
                    }
                }
            }
        }

        verificarVitoria() {
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
    }

    const campoMinado = new CampoMinado();
    let jogo = campoMinado.criarJogo();

    function iniciarJogo() {
        if (!nomeJogador.trim()) return;

        jogo = campoMinado.criarJogo();
        jogoIniciado = true;
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

    onMount(() => {
        const dados = localStorage.getItem('ranking-campo-minado');
        if (dados) {
            ranking = JSON.parse(dados);
        }
    });
</script>

<div class="tela-jogo">
    <h1 class="titulo-jogo">💣Tente não explodir💣</h1>

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
                                    campoMinado.revelarCelula(
                                        indiceLinha,
                                        indiceColuna
                                    )
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
                        <strong>{indice + 1}º</strong>
                        — {jogador.nome} ({jogador.pontos})
                    </li>
                {/each}
            </ul>

            <a class="button link-reset" href="/">
                Voltar ao Menu
            </a>
        </div>
    </div>
{/if}
