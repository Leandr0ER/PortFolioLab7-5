<svelte:head>
    <title>My page</title>
</svelte:head>

<script>
    import projects from "$lib/projects.json";
    import Project from "$lib/Project.svelte";

    import { onMount } from "svelte";

    let githubData = null;
    let loading = true;
    let error = null;

    onMount(async () => {
        try {
            const response = await fetch("https://api.github.com/users/Leandr0ER");
            githubData = await response.json();
        } catch (err) {
            error = err;
        }
        loading = false;
    });

</script>
<h1> Mike Corleone</h1>
   
<img src="./images/Imagen001.jpg" alt="mike" width="500px">
    
<p>Michael "Mike" Corleone is the protagonist of the 1972 Paramount Pictures crime film The Godfather and its 1974 sequel The Godfather Part II.
    Depicted as the youngest son of Vito Corleone, the head of a powerful Mafia family, Mike is a highly intelligent and initially reluctant participant in his family's criminal enterprise.
    Mike's world takes a dramatic turn when his father is targeted by rival Mafia families.
    Stepping up to protect his family, Mike becomes increasingly involved in their dangerous world, eventually uncovering betrayals and playing a key role in consolidating the Corleone family's power and solving internal conflicts.
</p>

<h2>Latest projects</h2>
<div class="projects">
{#each projects.slice(0, 3) as p}
    <Project data={p} hLevel="3" />
{/each}

{#if loading}
    <p>Loading...</p>
{:else if error}
    <p class="error">Something went wrong: {error.message}</p>
{:else}
    <section>
        <h2>My GitHub Stats</h2>
        <dl>
            <dt>Followers</dt>
            <dd>{githubData.followers}</dd>
            <dt>Following</dt>
            <dd>{githubData.following}</dd>
            <dt>Public Repositories</dt>
            <dd>{githubData.public_repos}</dd>
        </dl>
    </section>
{/if}


</div>
