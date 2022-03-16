<script>
import Map from './Map.svelte';
import Select from 'svelte-select';
let district;

let items = []
for (let i = 1; i<68; i++) {
	items.push({value: i, label: "Senate District " + i, group: "senate"}, {value: i+"A", label: "House District " + i + "A", group: "house"},  {value: i+"B", label: "House District " + i + "B", group: "house"})
}

const groupBy = (item) => item.group;

function handleSelect(event) {
	district = event.detail.value;
}

function handleClear() {
	district = "";
}


</script>	

<style>

</style>


<div class="m-form-item">Select a 2012 district:</div> <Select {items} on:select={handleSelect} on:clear={handleClear} {groupBy} />

<Map {district}/>