<script>
    import { onMount } from 'svelte'
    import { api_ip } from './config';
    import "./app.css";

    let data = {}
    let isOverlayOpen = false;
    let overlayMode = "";
    
    let addMediaOverlayCategory = "";
    let addMediaOverlayName = "";
    let addMediaOverlayError = "";

    let editMediaOverlayCategory = "";
    let editMediaOverlayName = "";
    let editMediaOverlayError = "";
    $: editMediaOverlayStatus = "";
    let editMediaOverlayDisplayName = ""; 

    function capitalizeFirstLetter(string) {
        return string.charAt(0).toUpperCase() + string.slice(1);
    }

    async function sendAddMedia() {
        if(addMediaOverlayName == "") {
            addMediaOverlayError = "Media name can't be empty!"
            return;
        }

        if(Object.keys(data[addMediaOverlayCategory]).includes(addMediaOverlayName.toLowerCase())) {
            addMediaOverlayError = `${addMediaOverlayName} already exists in ${addMediaOverlayCategory}`
            return;
        }

        const response = await fetch(`http://${api_ip}/api/v1/media`, {
            method: "PUT",
            headers: {
                "Content-Type": 'application/json',
            },
            body: JSON.stringify({
                "media": addMediaOverlayName,
                "category": addMediaOverlayCategory
            })
        });
        json = await response.json();
        console.log(json);

        fetchData();
    }

    async function sendRemoveMedia() {
        const response = await fetch(`http://${api_ip}/api/v1/media`, {
            method: "DELETE",
            headers: {
                "Content-Type": 'application/json',
            },
            body: JSON.stringify({
                "media": addMediaOverlayName,
                "category": addMediaOverlayCategory,
            })
        });
        json = await response.json();
        console.log(json);

        fetchData();
    }

    async function sendEditMedia() {
        const response = await fetch(`http://${api_ip}/api/v1/media`, {
            method: "PATCH",
            headers: {
                "Content-Type": 'application/json',
            },
            body: JSON.stringify({
                "media": editMediaOverlayName,
                "category": editMediaOverlayCategory,
                "disname": editMediaOverlayDisplayName,
                "status": editMediaOverlayStatus
            })
        });
        json = await response.json();
        console.log(json);

        fetchData();
    }

    function getAverageRating(category, media) {
        let totalRating = 0;
        let allSeasons = 0;
        Object.keys(data[category][media]).forEach((item, index) => {
            if(item != "status" && item != "disname") {
                allSeasons += 1;
                if(data[category][media][item]["rating"] != NaN)
                    totalRating += data[category][media][item]["rating"];
                else
                totalRating += 0;
            }
        })
        if(parseInt(totalRating / allSeasons).toString() != "NaN")
            return parseInt(totalRating / allSeasons).toString()
        else 
            return 0
    }

    onMount(async () =>  {
        fetchData()
    })

    async function fetchData() {
        const response = await fetch(`http://${api_ip}/api/v1/json`);
        data = await response.json();
        console.log(data);
    }
</script>

{#if isOverlayOpen}
    {#if overlayMode == "add"}
        <div id="overlay">
            <div id="overlay-content">
                <div id="overlay-close-button" on:click={() => {isOverlayOpen = false; addMediaOverlayError = ""; addMediaOverlayCategory = ""; addMediaOverlayName = ""}}>&times;</div>
                Add {addMediaOverlayCategory}
                <div id="overlay-add-button-form">
                    <input placeholder="Breaking Bad" bind:value={addMediaOverlayName}/>
                    <button on:click={() => sendAddMedia()}>
                        Add
                    </button>
                    <div id="overlay-error">
                        {addMediaOverlayError}
                    </div>
                </div>
            </div>
        </div>
    {:else if overlayMode == "remove"}
        <div id="overlay">
            <div id="overlay-content">
                <div id="overlay-close-button" on:click={() => {isOverlayOpen = false; addMediaOverlayError = ""; addMediaOverlayCategory = ""; addMediaOverlayName = ""}}>&times;</div>
                <div style="max-width: 480px;">
                    Are you sure you want to delete {data[addMediaOverlayCategory][addMediaOverlayName]["disname"]} from {addMediaOverlayCategory}?
                </div>
                <div id="overlay-add-button-form">
                    <button on:click={() => sendRemoveMedia()}>
                        Delete
                    </button>
                    <div id="overlay-error">
                        {addMediaOverlayError}
                    </div>
                </div>
            </div>
        </div>
    {:else if overlayMode == "edit"}
        <div id="overlay">
            <div id="overlay-content">
                <div id="overlay-close-button" on:click={() => {isOverlayOpen = false; editMediaOverlayError = ""; editMediaOverlayCategory = ""; editMediaOverlayName = ""}}>&times;</div>
                <div style="max-width: 480px;">
                    Edit the {editMediaOverlayCategory}, {editMediaOverlayDisplayName}
                </div>
                <div id="overlay-add-button-form">
                    <input placeholder="Display Name" bind:value={editMediaOverlayDisplayName}/>
                    <div style="display: flex; flex-direction: row; gap: 13px; margin-bottom: 10px">
                        {#each ["planned", "watching", "completed", "paused", "dropped"] as status}
                            {#if status != data[editMediaOverlayCategory][editMediaOverlayName]["status"]}
                                <div id="edit-status" on:click={() => {editMediaOverlayStatus = status}}>
                                    {capitalizeFirstLetter(status)}
                                </div>
                            {:else}
                                <div id="edit-status-selected">
                                    {capitalizeFirstLetter(status)}
                                </div>
                            {/if}
                        {/each}
                    </div>
                    <button on:click={() => sendRemoveMedia()}>
                        Edit
                    </button>
                    <div id="overlay-error">
                        {editMediaOverlayError}
                    </div>
                </div>
            </div>
        </div>
    {/if}
{/if}

{#key data}
    <main>
        {#each Object.keys(data) as category}
            <div id="category">
                <div id="category-title">
                    <h1>{capitalizeFirstLetter(category)}</h1>
                </div>
                {#each Object.keys(data[category]) as media}
                    <div id="media">
                        <div id="media-delete-button" on:click={() => {isOverlayOpen = true; overlayMode = "remove"; addMediaOverlayCategory = category; addMediaOverlayName = media}}>
                            <i class="gg-trash"></i>
                        </div>   
                        <div id="media-delete-button" style="margin-top: 20px" on:click={() => {isOverlayOpen = true; overlayMode = "edit"; editMediaOverlayCategory = category; editMediaOverlayName = media; editMediaOverlayStatus = data[category][media]["status"]; editMediaOverlayDisplayName = data[category][media]["disname"]}}>
                            <i class="gg-pen"></i>
                        </div>
                        
                        <div id="media-contents">
                            {data[category][media]["disname"]} <br/>
                        </div>
                        <div id="media-rating-box">
                            <div id="media-rating">
                                {getAverageRating(category, media)}
                            </div>
                        </div>
                    </div>
                {/each}
                <button id="add-button" on:click={() => {isOverlayOpen = true;  addMediaOverlayCategory = category; ; overlayMode = "add"}}>
                    <div id="add-button-contents">
                        Add {category}
                    </div>
                </button>
            </div>
        {/each}
    </main>
{/key}

<style lang="postcss">
@import url('https://css.gg/trash.css');
@import url('https://css.gg/pen.css');

    #edit-status {
        background-color: white;
        border: 2px solid #E9E8EC;
        border-radius: 5px;
        padding: 5px;
    }

    #edit-status-selected {
        background-color: #F6F5F8;
        border: 2px solid #E9E8EC;
        border-radius: 5px;
        padding: 5px;
    }

    #overlay {
        width: 100vw;
        height: 100vh;
        position: fixed;
        top: 0px;
        left: 0px;
        display: flex;
        justify-content: center;
        align-items: center;
    }

    #overlay-content {
        border: 2px solid #E9E8EC;
        border-radius: 10px;
        background-color: white;
        color: white;
        border-radius: 0.375rem;
        padding-left: 2rem;
        padding-right: 2rem;
        padding-top: 2.5rem;
        padding-bottom: 2.5rem;
        position: relative;
        max-width: 32rem;
        width: 500px;
        height: max-content;
        padding: 20px;
        color: #2c2858;
        font-size: 18px;
        overflow-wrap: break-word;
        font-weight: bold;

    }
    
    #overlay-close-button {
        position: absolute;
        top: 0.5rem;
        right: 0.75rem;
        font-size: 2.25rem;
        line-height: 2.5rem;
        color: rgb(209 213 219);
        transition-property: transform;
        transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1);
        transition-duration: 150ms;
    }

    #overlay-add-button-form {
        display: flex;
        flex-direction: column;
        gap: 5px;
        margin-top: 20px;
    }

    #overlay-error {
        color: red;
        font-weight: bold;
    }

    #overlay-close-button:hover {
        --tw-translate-y: -0.125rem/* -2px */;
    transform: translate(var(--tw-translate-x), var(--tw-translate-y)) rotate(var(--tw-rotate)) skewX(var(--tw-skew-x)) skewY(var(--tw-skew-y)) scaleX(var(--tw-scale-x)) scaleY(var(--tw-scale-y));
    }

    #media-delete-button {
        position: relative;
        font-size: 2.25rem;
        line-height: 2.5rem;
        color: black;
        margin-left: 215px;
        margin-top: 25px;
        transition: 0.2s;
        width: max-content;
        height: max-content;
    }

    #media-delete-button:hover {
        animation-name: cancel-button-hover;
        animation-timing-function: ease;
    }

    @keyframes cancel-button-hover {
        0% { transform: translateY(0) }
        100% { transform: translateY(10) }
    }

    #category {
        width: 290px;
        height: max-content;
        border-radius: 10px;
        background-color: #F6F5F8;
        display: flex;
        flex-direction: column;
        justify-content: center;
        gap: 20px;
        margin-bottom: 20px;
    }

    #category-title {
        padding-left: 20px;
    }

    #media {
        font-weight: bold;
        margin-left: auto;
        margin-right: auto;
        width: 250px;
        min-height: 100px;
        border: 2px solid #E9E8EC;
        border-radius: 10px;
        background-color: white;
    }

    #media-contents {
        margin-top: -65px;
        padding: 20px;
        color: #2c2858;
        font-size: 18px;
        max-width: 180px;
        overflow-wrap: break-word;
    }

    #media-rating-box {
        padding-left: 20px;
        padding-bottom: 20px;
    }

    #media-rating {

    }

    #add-button {
        font-weight: bold;
        margin-left: auto;
        margin-right: auto;
        width: 250px;
        height: max-content;
        border: 2px solid #e1e0e5;
        border-radius: 10px;
        background-color: #efeef1;
        margin-bottom: 20px;
        text-align: center;
        transition: 0.1s;
    }

    #add-button-contents {
        padding: 5px;
        color: #7c7989;
        font-size: 18px;
        overflow-wrap: break-word;
    }

    #add-button:hover {
        background-color: #E5E5E5;  
    }

    main {
        display: flex;
        flex-direction: row;
        gap: 30px;
        margin-left: 20px;
        margin-top: 20px;
    }
</style>