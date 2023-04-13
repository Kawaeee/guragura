<script>
    import Header from './Header.svelte';
    import Footer from './Footer.svelte';
    import Guragura from "./Guragura.svelte";
    import './styles.css';
</script>

<div class="app">
    <Header />

    <main>
        <Guragura />
    </main>

    <Footer />
</div>

<style>
    .app {
        display: flex;
        flex-direction: column;
        min-height: 100vh;
    }

    main {
        flex: 1;
        display: flex;
        flex-direction: column;
        padding: 1rem;
        width: 100%;
        max-width: 64rem;
        margin: 0 auto;
        box-sizing: border-box;
    }
</style>
