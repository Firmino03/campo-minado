<script lang="ts">
    import { goto } from '$app/navigation'

    class Coordenada {
        linha: number
        coluna: number
    }

    class EstadoJogo {
        mapa: number[][]     // 0 = vazio, 9 = mina
        revelado: boolean[][] // células reveladas
    }

    const LINHAS = 10
    const COLUNAS = 10
    const NUM_MINAS = 15


    function inicializarJogo(): EstadoJogo {
        let mapa = gerarMapaVazio(LINHAS, COLUNAS)
        espalharMinas(mapa, NUM_MINAS)
        gerarNumeros(mapa)

        let revelado = gerarMapaRevelado(LINHAS, COLUNAS)

        let jogo = new EstadoJogo()
        jogo.mapa = mapa
        jogo.revelado = revelado
        
        return jogo
    }

    function gerarMapaVazio(l: number, c: number): number[][] {
        return Array.from({ length: l }, () => Array(c).fill(0))
    }

    function gerarMapaRevelado(l: number, c: number): boolean[][] {
        return Array.from({ length: l }, () => Array(c).fill(false))
    }

    function espalharMinas(mapa: number[][], quant: number) {
        let colocadas = 0
        while (colocadas < quant) {
            const i = Math.floor(Math.random() * mapa.length)
            const j = Math.floor(Math.random() * mapa[0].length)

            if (mapa[i][j] !== 9) {
                mapa[i][j] = 9
                colocadas++
            }
        }
    }

    function gerarNumeros(mapa: number[][]) {
        for (let i = 0; i < mapa.length; i++) {
            for (let j = 0; j < mapa[0].length; j++) {
                if (mapa[i][j] === 9) continue

                let cont = 0
                for (let x = -1; x <= 1; x++) {
                    for (let y = -1; y <= 1; y++) {
                        let ni = i + x
                        let nj = j + y
                        if (
                            ni >= 0 && nj >= 0 &&
                            ni < mapa.length && nj < mapa[0].length &&
                            mapa[ni][nj] === 9
                        ) cont++
                    }
                }
                mapa[i][j] = cont
            }
        }
    }

    // Revelar célula
    function revelar(i: number, j: number) {
        if (jogo.revelado[i][j]) return
        jogo.revelado[i][j] = true

        // clicou em mina
        if (jogo.mapa[i][j] === 9) {
            alert("💥 Você explodiu!")
            goto("/")
        }

        // se for zero, revela vizinhos automaticamente
        if (jogo.mapa[i][j] === 0) {
            revelarVizinhos(i, j)
        }

        checarVitoria()
    }

    function revelarVizinhos(i: number, j: number) {
        for (let x = -1; x <= 1; x++) {
            for (let y = -1; y <= 1; y++) {
                let ni = i + x
                let nj = j + y

                if (ni >= 0 && nj >= 0 && ni < LINHAS && nj < COLUNAS) {
                    if (!jogo.revelado[ni][nj] && jogo.mapa[ni][nj] !== 9) {
                        revelar(ni, nj)
                    }
                }
            }
        }
    }

    function checarVitoria() {
        for (let i = 0; i < LINHAS; i++) {
            for (let j = 0; j < COLUNAS; j++) {
                if (jogo.mapa[i][j] !== 9 && !jogo.revelado[i][j]) {
                    return
                }
            }
        }
        alert("🎉 Você venceu!")
        goto("/")
    }

    let jogo: EstadoJogo = inicializarJogo()
</script>

<h1>💣Tente não explodir💣</h1>

<table>
    {#each jogo.mapa as linha, i}
        <tr>
            {#each linha as celula, j}
                <td
                    class="celula"
                    on:click={() => revelar(i, j)}
                >
                    {#if jogo.revelado[i][j]}
                        {celula === 9 ? "💣" : (celula === 0 ? "" : celula)}
                    {/if}
                </td>
            {/each}
        </tr>
    {/each}
</table>

<br />

<a class="menu" href="/">Voltar ao Menu</a>




