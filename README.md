# HTML
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Weather App</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>

    <div class="weather-app">
        <h1>Weather App</h1>
        <p>Check the weather of any city</p>

        <div class="search-box">
            <input
                type="text"
                id="cityInput"
                placeholder="Enter city name"
            >
            <button id="searchBtn">Search</button>
        </div>

        <p id="message">Enter a city to check its weather.</p>

        <div id="weatherResult" class="weather-result"></div>
    </div>

    <script src="script.js"></script>
</body>
</html>

# css


* {
    box-sizing: border-box;
}

body {
    margin: 0;
    font-family: Arial, sans-serif;
    background: linear-gradient(135deg, #60a5fa, #1e40af);
    min-height: 100vh;
    display: flex;
    justify-content: center;
    align-items: center;
    padding: 20px;
}

.weather-app {
    background: white;
    width: 100%;
    max-width: 430px;
    padding: 30px 24px;
    border-radius: 18px;
    text-align: center;
    box-shadow: 0 10px 30px #00000020;
}

h1 {
    color: #1e3a8a;
    margin-top: 0;
}

.search-box {
    display: flex;
    gap: 8px;
    margin: 25px 0;
}

input {
    width: 100%;
    min-width: 0;
    padding: 12px;
    border: 1px solid #cbd5e1;
    border-radius: 8px;
    font-size: 15px;
}

button {
    padding: 12px 16px;
    border: none;
    border-radius: 8px;
    background: #2563eb;
    color: white;
    cursor: pointer;
}

button:hover {
    background: #1d4ed8;
}

button:disabled {
    opacity: 0.6;
    cursor: not-allowed;
}

.weather-result {
    margin-top: 20px;
}

.temperature {
    font-size: 42px;
    font-weight: bold;
    color: #1e3a8a;
    margin: 12px 0;
}

.details {
    background: #eff6ff;
    padding: 12px;
    border-radius: 10px;
    margin-top: 12px;
}

.error {
    color: #dc2626;
}

.loading {
    color: #2563eb;
}

# JS


const cityInput = document.getElementById("cityInput");
const searchBtn = document.getElementById("searchBtn");
const message = document.getElementById("message");
const weatherResult = document.getElementById("weatherResult");

async function getWeather() {
    const city = cityInput.value.trim();

    if (city === "") {
        message.textContent = "Please enter a city name.";
        message.className = "error";
        weatherResult.innerHTML = "";
        return;
    }

    try {
        searchBtn.disabled = true;
        message.textContent = "Loading weather...";
        message.className = "loading";
        weatherResult.innerHTML = "";

        const cityResponse = await fetch(
            `https://geocoding-api.open-meteo.com/v1/search?name=${encodeURIComponent(city)}&count=1&language=en&format=json`
        );


        const cityData = await cityResponse.json();

        if (!cityData.results || cityData.results.length === 0) {
            throw new Error("City not found. Try another name.");
        }

        const location = cityData.results[0];

        const weatherResponse = await fetch(
            `https://api.open-meteo.com/v1/forecast?latitude=${location.latitude}&longitude=${location.longitude}&current=temperature_2m,relative_humidity_2m,apparent_temperature,weather_code,wind_speed_10m&timezone=auto`
        );


        const weatherData = await weatherResponse.json();
        const current = weatherData.current;

        const code = current.weather_code;
        let condition = "Unknown";

        if (code === 0) {
            condition = "Clear sky ☀️";
        } else if (code <= 3) {
            condition = "Cloudy ⛅";
        } else if (code <= 48) {
            condition = "Foggy 🌫️";
        } else if (code <= 67) {
            condition = "Rainy 🌧️";
        } else if (code <= 77) {
            condition = "Snowy ❄️";
        } else if (code <= 82) {
            condition = "Rain showers 🌦️";
        } else if (code <= 86) {
            condition = "Snow showers 🌨️";
        } else if (code >= 95) {
            condition = "Thunderstorm ⛈️";

        weatherResult.innerHTML = `
            <h2>${condition}</h2>

            <div class="temperature">
                ${current.temperature_2m}°C
            </div>

            <p>Feels like: ${current.apparent_temperature}°C</p>

            <div class="details">
                <p>💧 Humidity: ${current.relative_humidity_2m}%</p>
                <p>💨 Wind Speed: ${current.wind_speed_10m} km/h</p>
            </div>
        `;

    } catch (error) {
        message.textContent = error.message.includes("City not found")
            ? error.message
            : "Unable to load weather. Check your internet and try again.";

        message.className = "error";
        weatherResult.innerHTML = "";

    } finally {
        searchBtn.disabled = false;
    }
}

searchBtn.addEventListener("click", getWeather);

cityInput.addEventListener("keydown", function(event) {
    if (event.key === "Enter") {
        getWeather();
    }
});
