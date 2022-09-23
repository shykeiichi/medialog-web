<script>
  import { onMount } from "svelte";
  import { get } from "svelte/store";
	import data from "./Data.json"

	let statuses = ["planned", "watching", "completed", "paused", "dropped"];
	let categories = ["series", "movie", "book", "podcast"];

    function capitalizeFirstLetter(string) {
        return string.charAt(0).toUpperCase() + string.slice(1);
    }

	function cutOffString(string) {
		if(string.length <= 15) {
			return string;
		}
		
		return string.slice(0, 13) + "...";
	}

	function containsAnyFromArray(string, array) {
		if(array.length == 0)
			return true;

		let ret = "false";
		array.forEach((value) => {
			// console.log(string.includes(value));
			// console.log(`${string} ${value} ${string.includes(value)}`);
			if(string.includes(value)) {
				ret = "true";
			}
		});
		if(ret == "true")
			return true;
		else 
			return false;
	}

	function getRanomUnwatched() {
		let allshows = []
		let allshowscat = []

		categories.forEach((category) => {
			if(containsAnyFromArray(category, FilterFormat)) {
				Object.keys(data[category]).forEach((media) => {
					if(containsAnyFromArray(data[category][media]["status"], FilterStatus)) {
						allshows.push(media);
						allshowscat.push(category);
					}
				});
			}
		})

		let random = allshows[Math.floor(Math.random()*allshows.length)];
		return data[allshowscat[allshows.indexOf(random)]][random]["disname"];
	}


	// function getVals(){
	// 	// Get slider values
	// 	var parent = this.parentNode;
	// 	var slides = parent.getElementsByTagName("input");
	// 	var slide1 = parseFloat( slides[0].value );
	// 	var slide2 = parseFloat( slides[1].value );
	// 	// Neither slider will clip the other, so make sure we determine which is larger
	// 	if( slide1 > slide2 ){ var tmp = slide2; slide2 = slide1; slide1 = tmp; }

	// 	return [slide1, slide2]
	// }

	// window.onload = function(){
	// 	// Initialize Sliders
	// 	var sliderSections = document.getElementsByClassName("range-slider");
	// 		for( var x = 0; x < sliderSections.length; x++ ){
	// 			var sliders = sliderSections[x].getElementsByTagName("input");
	// 			for( var y = 0; y < sliders.length; y++ ){
	// 				if( sliders[y].type ==="range" ){
	// 					sliders[y].oninput = getVals;
	// 					// Manually trigger event first time to display values
	// 					sliders[y].oninput();
	// 				}
	// 			}
	// 		}
	// }

	let FilterName = "";
	let FilterFormat = [];
	let FilterStatus = "";
	let FilterRatingMin = 0;
	let FilterRatingMax = 100;

	function updateLocalStorage() {
		localStorage.setItem("filter", JSON.stringify({
			"name": FilterName,
			"format": FilterFormat,
			"status": FilterStatus,
			"ratingmin": FilterRatingMin,
			"ratingmax": FilterRatingMax
		}))
	}

	function pullLocalStorage() {
		filterstorage = JSON.parse(localStorage.getItem("filter"))
	
		FilterName = filterstorage["name"];
		FilterFormat = filterstorage["format"];
		FilterStatus = filterstorage["status"];
		FilterRatingMin = filterstorage["ratingmin"];
		FilterRatingMax = filterstorage["ratingmax"];
	}

	onMount(() => {
		pullLocalStorage();
	});

	let PickGetRandom = "Get Random";
</script>

<div class="FilterContainer">
	<div class="MediaHeader">
		Pick
	</div>
	<div class="FilterBox">
		<button on:click={() => {PickGetRandom = getRanomUnwatched()}}> 
			{PickGetRandom}
		</button>
	</div>
	<div class="MediaHeader">
		Filter
	</div>
	<div class="FilterBox">
		Name
		<input bind:value={FilterName} placeholder="Name" />
		Category
		<select bind:value={FilterFormat} on:change={() => updateLocalStorage()} multiple>
			{#each categories as category}
				<option value={category}>{capitalizeFirstLetter(category)}</option>
			{/each}
		</select>
		Status
		<select bind:value={FilterStatus} on:change={() => updateLocalStorage()} multiple>
			{#each statuses as status}
				<option value={status}>{capitalizeFirstLetter(status)}</option>
			{/each}
		</select>
		Rating {FilterRatingMin}-{FilterRatingMax}
		<section class="range-slider">
			<span class="rangeValues"></span>
			<input bind:value={FilterRatingMin} on:change={() => {updateLocalStorage(); FilterRatingMin = FilterRatingMin > FilterRatingMax ? FilterRatingMax : FilterRatingMin}} min="0" max="100" step="1" type="range">
			<input bind:value={FilterRatingMax} on:change={() => {updateLocalStorage(); FilterRatingMax = FilterRatingMax < FilterRatingMin ? FilterRatingMin : FilterRatingMax}} min="0" max="100" step="1" type="range">
		</section>
		<button on:click={() => {
			FilterFormat = [];
			FilterName = "";
			FilterStatus = "";
		}} on:change={() => updateLocalStorage()}>Clear Filter</button>
	</div>
</div>
<div class="App">
	{#each statuses as status}
		{#if containsAnyFromArray(status, FilterStatus)}
			<div class="MediaHeader">
				{capitalizeFirstLetter(status)}
			</div>
			<div class="MediaContainer">
				<table>
					<thead>
						<tr id="MediaItem">
							<td id="MediaItemIcon" />
							<td id="MediaItemTitle">
								Title
							</td>
							<td id="MediaItemMisc">
								Rating
							</td>
							<td id="MediaItemMisc">
								Studio
							</td>
							<td id="MediaItemMisc">
								Type
							</td>
						</tr>
					</thead>
					<tbody>
					{#each Object.keys(data) as category}
						{#each Object.keys(data[category]) as media}
							{#each Object.keys(data[category][media]["seasons"]) as season}
								{#if data[category][media]["status"] == status && `${media.toLowerCase()} `.includes(FilterName.toLowerCase())}
									{#if containsAnyFromArray(category, FilterFormat) && data[category][media]["seasons"][season]["rating"] >= FilterRatingMin && data[category][media]["seasons"][season]["rating"] <= FilterRatingMax}
										<tr id="MediaItem">
											<td id="MediaItemIcon" >
												<img id="MediaItemIconImg" src="./coverimages/{category}/{media} {season}.jpg" alt="cover" onerror="this.onerror=null; this.src='Default.png'">
											</td>
											<td id="MediaItemTitle">
												{#if Object.keys(data[category][media]["seasons"]).length == 1}
													{data[category][media]["disname"]}
												{:else}	
													{data[category][media]["disname"]} <i>{capitalizeFirstLetter(season)}</i>
												{/if}
											</td>
											<td id="MediaItemMisc">
												{data[category][media]["seasons"][season]["rating"]}
											</td>
											<td id="MediaItemMisc">
												{cutOffString(capitalizeFirstLetter(data[category][media]["seasons"][season]["studio"]))}
											</td>
											<td id="MediaItemMisc">
												{capitalizeFirstLetter(category)}
											</td>
										</tr>
									{/if}
								{/if}
							{/each}
						{/each}
					{/each}
					</tbody>
				</table>
			</div>
		{/if}
	{/each}
	<div class="Spacer">
		asdasd
	</div>
</div>


<style>

.Spacer {
	height: 200px;
	width: 10vw;
	color: rgba(255, 255, 255, 0);
}

.FilterContainer {
	position: absolute;
	top: 20px;
	left: 20px;
}

.FilterBox {
	background-color: white;
	border-radius: 5px;
	padding: 10px;
	filter: drop-shadow(0px 0px 5px rgb(223, 223, 223));
	font-size: 16px;
	margin-top: 10px;
	margin-bottom: 10px;
	display: flex;
	flex-direction: column;
	gap: 5px;
}

.App {
  width: 50%;
  height: 100vh;
  display: flex;
  flex-direction: column;
  margin-left: auto;
  margin-right: auto;
  padding-top: 20px;
  gap: 10px;
}

.MediaHeader {
  padding-left: 20px;
  font-size: 22px;
  color:slategrey;
  font-weight: lighter;
}

.MediaContainer {
  background-color: white;
  width: 100%;
  border-radius: 5px;
  padding: 10px;
  filter: drop-shadow(0px 0px 5px rgb(223, 223, 223));
  font-size: 16px;
  margin-bottom: 10px;
}

#MediaItemIconImg {
	width: 35px;
  	height: 50px;
	border-radius: 5px;
}

#MediaItemIcon {
	width: 50px;
	text-align: center;
}

#MediaItemTitle {
  width: auto;
}

#MediaItemMisc {
  width: 120px;
  text-align: center;
}

#MediaItem {
  width: 100%;
  padding-bottom: 20px;
}

table {
  width: 100%;
  border-spacing: 5px;
  color: #43546e;
}

thead {
	font-weight: bold;
	color: #707982;
}

section.range-slider {
    position: relative;
    width: 200px;
    height: 35px;
    text-align: center;
}

section.range-slider input {
    pointer-events: none;
    position: absolute;
    overflow: hidden;
    left: 0;
    top: 15px;
    width: 200px;
    outline: none;
    height: 18px;
    margin: 0;
    padding: 0;
}

section.range-slider input::-webkit-slider-thumb {
    pointer-events: all;
    position: relative;
    z-index: 1;
    outline: 0;
}

section.range-slider input::-moz-range-thumb {
    pointer-events: all;
    position: relative;
    z-index: 10;
    -moz-appearance: none;
    width: 9px;
}
</style>