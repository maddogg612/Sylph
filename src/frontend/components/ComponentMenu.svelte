<script>
import { flip } from 'svelte/animate';
  import { dndzone, TRIGGERS, SHADOW_ITEM_MARKER_PROPERTY_NAME } from 'svelte-dnd-action';
 

  export let items;
  export let nodes;
  const flipDurationMs = 300;
  let shouldIgnoreDndEvents = false;
  let dropFromOthersDisabled = true;


  // code courtesy of https://svelte.dev/repl/924b4cc920524065a637fa910fe10193?version=3.24.1
  function handleDndConsider(e) {
      // console.warn(`got consider ${JSON.stringify(e.detail, null, 2)}`);
      const {trigger, id} = e.detail.info;
      if (trigger === TRIGGERS.DRAG_STARTED) {
        // console.warn(`copying ${id}`);
        const idx = items.findIndex(item => item.id === id);
        const newId = `${id}_copy_${Math.round(Math.random() * 100000)}`;
        // the line below was added in order to be compatible with version svelte-dnd-action 0.7.4 and above 
        e.detail.items = e.detail.items.filter(item => !item[SHADOW_ITEM_MARKER_PROPERTY_NAME]);
        e.detail.items.splice(idx, 0, { ...JSON.parse(JSON.stringify(items[idx])), id: newId })
        // {...items[idx], id: newId});
        items = e.detail.items;
        shouldIgnoreDndEvents = true;
      }
      else if (!shouldIgnoreDndEvents) {
        items = e.detail.items;
      }
      else {
        items = [...items];
      }
  }
  function handleDndFinalize(e) {
    // console.warn(`got finalize ${JSON.stringify(e.detail, null, 2)}`);
    if (!shouldIgnoreDndEvents) {
      items = e.detail.items;
    }
    else {
        items = [...items];
        shouldIgnoreDndEvents = false;
    }
  }

  let counter = 0;
  let newComponent = ()=>{return{
    id: `node_test${++counter}`,
    name: 'test',
    items: [],
    attributes: {},
    styles: {},
    selected: false,}
  }

</script>

<style>
	section {
    box-sizing: border-box;
    display: flex;
    flex-direction: column;
    align-items: center;
		width: 100%;
    /* height: 100%; */
		padding: 0.3em;
    background-color: white;
	}
  
	div {
    box-sizing: border-box;
		width: 80%;
    padding: 0.2em;
		border: 1px solid blue;
		margin: 0.15em 0;
	}
</style>

<section 
  use:dndzone={{items, flipDurationMs, dropFromOthersDisabled }}
  on:consider={handleDndConsider} 
  on:finalize={handleDndFinalize}
>
  {#each items as item(item.id)}
		<div animate:flip="{{duration: flipDurationMs}}">
			{item.name}	
		</div>
	{/each}
</section>
<!-- <button on:click={()=>items = [...items, newComponent()]}>Add Component</button> -->
