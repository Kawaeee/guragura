<script>
    import github from '$lib/images/github.svg';
    import logo from '$lib/images/logo.png';
</script>

<header>
    <div class="logo">
        <a href="https://kawaeee.github.io/guragura/">
            <img src={logo} alt="Guragura"/>
        </a>
    </div>

    <div class="corner">
        <a href="https://github.com/Kawaeee/guragura">
            <img src={github} alt="GitHub" />
        </a>
    </div>
</header>

<style>
    header {
        display: flex;
        justify-content: space-between;
        align-items: center;
    }

    .logo {
        width: 10em;
        height: 3em;
        padding: 0 1em;
    }

    .logo a {
        display: flex;
        align-items: center;
        height: 100%;
    }

    .logo img {
        width: 100%;
        height: auto;
        object-fit: contain;
        background: transparent;
    }

    .corner {
        width: 3em;
        height: 3em;
    }

    .corner a {
        display: flex;
        align-items: center;
        justify-content: center;
        width: 100%;
        height: 100%;
    }

    .corner img {
        width: 2em;
        height: 2em;
        object-fit: contain;
    }

    a:hover {
        color: var(--color-theme-1);
    }
</style>
