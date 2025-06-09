<script lang="ts">
	let { children } = $props();
</script>

<div class="navbar" id="header">
    <div class="navbuttons">
        <a href="./" class="navbutton">Home</a>
        <a href="./" class="navbutton">Projects</a>
        <a href="./" class="navbutton">Links</a>
    </div>
</div>

<style>
    :global(body) {
        background-color: black;
        color: white;
        font-family: "Lucida Console", monospace;
        box-sizing: border-box;
    }
    :global(.bg) {
        position: absolute;
        top: 0;
        left: 0;
        width: 100vw;
        height: 100vh;
    }
    .navbar {
        display: flex;
        width: 100%;
        height: 50px;
        position: absolute;
        top: 0;
        left: 0;
        background-color: rgb(25, 13, 36);
        border-bottom: 8px solid rgb(35, 23, 46);
        overflow: hidden;
        z-index: 1000;
    }
    .navbuttons {
        position: relative;
        width: auto;
        height: 25px;
        margin: auto;
        overflow: hidden;
        /*background-color: white;*/
    }
    .navbutton {
        margin-left: 16px;
        margin-right: 16px;
        font-size: 25px;
        text-decoration: none;
        color: rgb(180, 180, 180);
    }
</style>

{@render children()}