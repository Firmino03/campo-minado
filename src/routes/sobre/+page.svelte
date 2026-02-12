<main class="sobre-container">
    <h1>Sobre</h1>
    <!-- BOTÕES -->
    <div class="botoes-sobre">
        <a href="https://github.com/Firmino03/campo-minado.git" target="_blank" class="link-reset">
            <button class="button">Nosso GitHub</button>
        </a>

        <a href="/" class="link-reset">
            <button class="button">Voltar ao Menu</button>
        </a>
    </div>

    <!-- GRID SOBRE O PROJETO -->
    <div class="card">
        <p class="card-text">
            O projeto é um jogo Campo Minado desenvolvido com Svelte, utilizando HTML, CSS e JavaScript. O framework foi utilizado para organizar os componentes e controlar o estado da aplicação de forma reativa.
        </p>

        <p class="card-text">
            A lógica do jogo consiste na geração automática de um tabuleiro com minas distribuídas aleatoriamente. Ao clicar em uma célula, o sistema verifica se há mina ou calcula a quantidade de minas ao redor. Caso não haja minas próximas, as células vizinhas são reveladas automaticamente. O jogador vence ao abrir todas as casas seguras e perde ao clicar em uma mina.
        </p>

        <p class="card-text">
            O sistema de ranking foi desenvolvido utilizando Local Storage, permitindo salvar localmente o nome do jogador e seu desempenho, mantendo os dados mesmo após atualizar ou fechar o navegador.
        </p>
    </div>

    <!-- GRID DESENVOLVEDORES -->
    <div class="card">
        <h2>Desenvolvedores</h2>

        <ul class="lista-devs">
            <li>Karolayne Firmino</li>
            <li>Maria Luiza</li>
        </ul>
	        <img class="sobre-logo" src="/images/logooifpe.png" alt="Logo IFPE" />

        <p class="curso">Curso de Informática Para Internet – IPI</p>
    </div>

</main>
