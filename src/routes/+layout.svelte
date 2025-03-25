<script>
import { base } from '$app/paths';
let pages = [
  { url: "/{base}", title: "Home" },
  { url: "{base}/projects", title: "Projects" },
  { url: "{base}/contact", title: "Contact" },
  { url: "{base}/resume", title: "Resume" },
  { url: "https://github.com/Mike-Kowalski", title: "Github" }
];
</script>
<nav>
  {#each pages as p}
  <a href="{p.url}">{p.title}</a>
  {/each}
</nav>
<slot />
