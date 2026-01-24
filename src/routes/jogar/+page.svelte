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
            const linhaAleatoria = Math.floor(Math.random() * tabuleiro.length);
            const colunaAleatoria = Math.floor(Math.random() * tabuleiro[0].length);

            if (tabuleiro[linhaAleatoria][colunaAleatoria] !== 9) {
                tabuleiro[linhaAleatoria][colunaAleatoria] = 9;
                minasColocadas++;
            }
        }
    }

    function calcularNumeros(tabuleiro) {
        for (let linha = 0; linha < tabuleiro.length; linha++) {
            for (let coluna = 0; coluna < tabuleiro[0].length; coluna++) {
                if (tabuleiro[linha][coluna] === 9) continue;

                let minasAoRedor = 0;

                for (let dLinha = -1; dLinha <= 1; dLinha++) {
                    for (let dColuna = -1; dColuna <= 1; dColuna++) {
                        const linhaVizinha = linha + dLinha;
                        const colunaVizinha = coluna + dColuna;

                        if (
                            linhaVizinha >= 0 &&
                            colunaVizinha >= 0 &&
                            linhaVizinha < tabuleiro.length &&
                            colunaVizinha < tabuleiro[0].length &&
                            tabuleiro[linhaVizinha][colunaVizinha] === 9
                        ) {
                            minasAoRedor++;
                        }
                    }
                }

                tabuleiro[linha][coluna] = minasAoRedor;
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
            revelarCelulasVizinhas(linha, coluna);
        }

        verificarVitoria();
    }

    function revelarCelulasVizinhas(linhaCentral, colunaCentral) {
        for (let dLinha = -1; dLinha <= 1; dLinha++) {
            for (let dColuna = -1; dColuna <= 1; dColuna++) {
                const linhaVizinha = linhaCentral + dLinha;
                const colunaVizinha = colunaCentral + dColuna;

                if (
                    linhaVizinha >= 0 &&
                    colunaVizinha >= 0 &&
                    linhaVizinha < TOTAL_LINHAS &&
                    colunaVizinha < TOTAL_COLUNAS &&
                    !jogo.celulasReveladas[linhaVizinha][colunaVizinha] &&
                    jogo.tabuleiro[linhaVizinha][colunaVizinha] !== 9
                ) {
                    revelarCelula(linhaVizinha, colunaVizinha);
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
        const novoRegistro = {
            nome: nomeJogador,
            pontos: pontuacao
        };

        ranking.push(novoRegistro);
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

<h1>💣 Tente não explodir 💣</h1>

{#if !jogoIniciado}
    <div class="container-inicio">
        <div class="card-inicio-jogo">
            <h2 class="titulo-fim">Digite seu nome</h2>

            <input
                class="input-nome"
                placeholder="Seu nome"
                bind:value={nomeJogador}
            />

            <button class="botao-menu" on:click={iniciarJogo}>
                Iniciar jogo
            </button>
        </div>
    </div>
{:else}
    <table>
        {#each jogo.tabuleiro as linhaTabuleiro, linha}
            <tr>
                {#each linhaTabuleiro as celula, coluna}
                    <td
                        class="celula"
                        on:click={() => revelarCelula(linha, coluna)}
                    >
                        {#if jogo.celulasReveladas[linha][coluna]}
                            {celula === 9 ? "💣" : celula === 0 ? "" : celula}
                        {/if}
                    </td>
                {/each}
            </tr>
        {/each}
    </table>
{/if}

{#if fimDeJogo}
    <div class="overlay">
        <div class="card-inicio-jogo">
            <h2 class="titulo-fim">
                {resultadoJogo === 'derrota'
                    ? '💥 Você explodiu!'
                    : '🎉 Você venceu!'}
            </h2>

            <p class="texto-fim">Ranking</p>

                <ul class="ranking">
            {#each ranking as jogador, index}
            <li>
            <strong>{index + 1}º</strong> — {jogador.nome}
            ({jogador.pontos} pontos)
            </li>
        {/each}
        </ul>


            <a class="botao-menu" href="/">Voltar ao Menu</a>
        </div>
    </div>
{/if}
