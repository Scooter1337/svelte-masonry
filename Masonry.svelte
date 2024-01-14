<script lang="ts">
  import { onMount, tick } from "svelte";
  export let items: any = []; // pass in data if it's dynamically updated
  let grids: grid[] = [],
    masonryElement: HTMLDivElement | undefined;

  type grid = {
    _el: HTMLDivElement;
    gap: number;
    items: HTMLElement[];
    ncol: number;
    mod: number;
  };

  export let reset: boolean;
  $: if (reset) {
    masonryElement = masonryElement;
  }

  export const refreshLayout = async () => {
    grids.forEach(async (grid: grid) => {
      /* get the post relayout number of columns */
      let ncol = getComputedStyle(grid._el).gridTemplateColumns.split(
        " "
      ).length;

      grid.items.forEach((c: HTMLElement) => {
        let new_h = c.getBoundingClientRect().height;

        if (new_h !== +c.dataset.h) {
          c.dataset.h = new_h.toString();
          grid.mod++;
        }
      });

      /* if the number of columns has changed */
      if (grid.ncol !== ncol || grid.mod) {
        /* update number of columns */
        grid.ncol = ncol;
        /* revert to initial positioning, no margin */
        grid.items.forEach((c: HTMLElement) =>
          c.style.removeProperty("margin-top")
        );
        /* if we have more than one column */
        if (grid.ncol > 1) {
          grid.items.slice(ncol).forEach((c: HTMLElement, i: number) => {
            let prev_fin =
                grid.items[i].getBoundingClientRect()
                  .bottom /* bottom edge of item above */,
              curr_ini =
                c.getBoundingClientRect().top; /* top edge of current item */

            c.style.marginTop = `${prev_fin + grid.gap - curr_ini}px`;
          });
        }

        grid.mod = 0;
      }
    });
  };

  const calcGrid = async (_masonryArr: HTMLDivElement[]) => {
    await tick();
    if (
      _masonryArr.length &&
      getComputedStyle(_masonryArr[0]).gridTemplateRows !== "masonry"
    ) {
      grids = _masonryArr.map((grid) => {
        return {
          _el: grid,
          gap: parseFloat(getComputedStyle(grid).rowGap),
          items: [...grid.childNodes].filter(
            (c: Element) =>
              c.nodeType === 1 && +getComputedStyle(c).gridColumnEnd !== -1
          ),
          ncol: 0,
          mod: 0,
        };
      }) as grid[];
      refreshLayout(); /* initial load */
    }
  };

  let _window: typeof window;
  onMount(() => {
    _window = window;
    _window.addEventListener("resize", refreshLayout); /* on resize */

    return () => {
      if (_window)
        _window.removeEventListener("resize", refreshLayout); /* on resize */
    };
  });

  $: if (masonryElement) {
    calcGrid([masonryElement]);
  }

  $: if (items) {
    // update if items are changed
    masonryElement = masonryElement; // refresh masonryElement
  }

  let className: string | undefined | null = "grid";

  export { className as class };
</script>

<div bind:this={masonryElement} class={className}>
  <slot />
</div>

<!-- 
  An almost direct copy and paste of: https://css-tricks.com/a-lightweight-masonry-solution
  
  Usage:
    - stretchFirst stretches the first item across the top

  <Masonry stretchFirst={true} >
    {#each data as o}
      <div class="_card _padding">
        Here's some stuff {o.name}
        <header>
          <h3>{o.name}</h3>
        </header>
        <section>
          <p>{o.text}</p> 
        </section>
      </div>
    {/each}
  </Masonry>
 -->
