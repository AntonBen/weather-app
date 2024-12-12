<script lang="js">
    import Rain from '$lib/rain.svelte';
    import Sun from '$lib/Sun.svelte';
    import { onMount } from 'svelte';

    let inputCity
    let selectedCity = {
        "latitude": 59.3294,
        "longitude": 18.0687,
        "name": "Stockholm"
    }
    let weathers = $state({});

    async function getCity(name) {
        try{
            const res = await fetch(`https://geocoding-api.open-meteo.com/v1/search?name=${name}&count=10&language=en&format=json`);
            const cities = await res.json();
            const city = {
                latitude: cities.results[0].latitude,
                longitude: cities.results[0].longitude,
                name: cities.results[0].name,
            }

            return city
        }
        catch(e) {
            console.log(e)
        }
	}

    async function getWeather(city) {
        try{
            const params = new URLSearchParams({
                "latitude": city.latitude,
                "longitude": city.longitude,
                "daily": ["weather_code", "temperature_2m_max"]
            });
            const res = await fetch('https://api.open-meteo.com/v1/forecast?' + params);
            const weather = await res.json();

            return weather.daily;
        }
        catch(e) {
            console.log(e)
        }
	}

    async function searchWeather(cityName) {
        selectedCity = await getCity(cityName);
        weathers = await getWeather(selectedCity);
    }

    function getDayName(dateStr)
    {
        var date = new Date(dateStr);
        return date.toLocaleDateString("EN", { weekday: 'long' });        
    }

    onMount(async () => {
        weathers = await getWeather(selectedCity);
    });

</script>

<main-body>
    <main-header>
        <input bind:value={inputCity} placeholder="Enter a city...">
        <button on:click={() => searchWeather(inputCity)}>
            Search
        </button>
    </main-header>
    <main-window>
        {#if weathers?.time?.length > 0}
            <Sun/>
            <text-main>
                <h2>{weathers.temperature_2m_max[0]}°C</h2>
                <h3>{selectedCity.name}</h3>
                <time>Sunday 17:00</time>
            </text-main>
        {/if}
    </main-window>
    <main-footer>
        {#each weathers.temperature_2m_max as temp, i }
            {#if i !== 0}
                <small-weather>
                    <svg-wrapper>
                        <Rain />
                    </svg-wrapper>
                    <day>{getDayName(weathers.time[i])}</day>
                    <temp>{temp}°C</temp>
                </small-weather>
            {/if}
        {/each}
    </main-footer>
</main-body>

<style>
    input {
        height: 44px;
        max-width: 296px;
        width: 100%;
        border-radius: 29.22px;
        padding: 0px 18px;
        box-sizing: border-box;
    }

    input::placeholder {
        opacity: 0.5;
    }

    button {
        height: 44px;
        border-radius: 29.22px;
        max-width: 188px;
        width: 100%;
        background-color: #1090DB;
        color:white;
        border: none;
        padding: 0;
        cursor: pointer;
        outline: inherit;
    }

    h2 {
        margin: 10px 0px;
        font-size: 44px;
        font-weight: 700;
    }

    h3 { 
        margin: 0px;
        margin-bottom: 8px;
    }

    main-body {
        display: flex;
        flex-direction: column;
        flex: 1;
        height: 100%;
        padding: 48px 56px;
        font-family: sans-serif;
        max-width: 830px;
    }

    main-footer {
        color:white;
        display: flex;
        gap: 48px;
        overflow: auto;
    }

    main-header {
        gap: 16px;
        display: flex;
    }

    main-window {
        display: flex;
        padding: 56px 0px;
        gap: 56px;
        border-bottom: solid 1px #ffffff7b;
        max-width: 700px;
        margin-bottom: 30px;
    }

    text-main {
        color:white;
        display: flex;
        flex-direction: column;
    }

    small-weather {
        width: 75px;
        display: flex;
        flex-direction: column;
        align-items: center;
    }

    @media (max-width: 640px) {
        button, input {
            max-width: 100%;
            margin-bottom: 16px;
        }

        main-body {
            padding: 40px 36px;
        }

        main-footer {
            display: flex;
            flex-direction: column;
            gap:0px;
        }

        small-weather {
            display: flex;
            flex-direction: row;
            gap: 14px;
            border-top: solid 1px #ffffff7b;
            width: 100%;
            padding: 14px 0px;
        }

        small-weather > svg {
            flex-grow: 1;
        }

        temp {
            justify-self: flex-end;
            font-size: 20px;
            font-weight: 500;
        }

        day {
            flex-grow: 1;
        }

        main-header {
            display: block;
        }

        main-window {
            flex-direction: column;
            gap: 40px;
            margin: 0px;
            padding: 40px 0px;
            border-bottom: none;
        }
    }
</style>