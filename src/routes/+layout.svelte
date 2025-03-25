<script>
import { base } from '$app/paths';  
alert(`Base path: ${base}`);
let pages = [
  { url: "/", title: "Home" },
  { url: "/projects", title: "Projects" },
  { url: "/contact", title: "Contact" },
  { url: "/resume", title: "Resume" },
  { url: "https://github.com/Mike-Kowalski", title: "Github" }
];
</script>
<nav>
  {#each pages as p}
  <a href={p.url.startsWith("http") ? p.url : ${base}${p.url}}
    class:current={$page.route.id === p.url}
    target={p.url.startsWith("http") ? "_blank" : undefined}
  >
    {p.title}
  </a>
  {/each}
</nav>
<slot />
