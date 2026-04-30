<!DOCTYPE html>
<html>
<head>
    <title>Pro Weather App</title>
    <style>
        body {
            font-family: Arial;
            text-align: center;
            padding: 40px;
            background: linear-gradient(to right, #4facfe, #00f2fe);
            color: white;
            transition: 0.3s;
        }

        .dark {
            background: #1e1e1e;
            color: white;
        }

        .box {
            background: rgba(0,0,0,0.3);
            padding: 20px;
            border-radius: 15px;
            display: inline-block;
        }

        button, input {
            padding: 10px;
            margin: 5px;
            border: none;
            border-radius: 5px;
        }

        button {
            background: orange;
            color: white;
            cursor: pointer;
        }

        img {
            width: 80px;
        }

        .forecast {
            display: flex;
            justify-content: center;
            gap: 10px;
            margin-top: 15px;
        }

        .card {
            background: rgba(255,255,255,0.2);
            padding: 10px;
            border-radius: 10px;
        }
    </style>
</head>
<body>

<div class="box">
    <h1>🌦 Weather App</h1>

    <input id="city" placeholder="Enter city">
    <br>
    <button onclick="getWeather()">Search</button>
    <button onclick="getLocationWeather()">📍 My Location</button>
    <button onclick="toggleDark()">🌙 Toggle Mode</button>

    <div id="result"></div>
    <div class="forecast" id="forecast"></div>
</div>

<script>
const API_KEY = "YOUR_API_KEY"; // 🔴 Replace this

function icon(num){
    return `https://developer.accuweather.com/sites/default/files/${num.toString().padStart(2,'0')}-s.png`;
}

async function getLocation(city){
    let res = await fetch(`https://dataservice.accuweather.com/locations/v1/cities/search?apikey=${API_KEY}&q=${city}`);
    let data = await res.json();
    return data[0];
}

async function getWeatherData(key){
    let res = await fetch(`https://dataservice.accuweather.com/currentconditions/v1/${key}?apikey=${API_KEY}`);
    let data = await res.json();
    return data[0];
}

async function getForecast(key){
    let res = await fetch(`https://dataservice.accuweather.com/forecasts/v1/daily/5day/${key}?apikey=${API_KEY}&metric=true`);
    let data = await res.json();
    return data.DailyForecasts;
}

async function showWeather(loc){
    let w = await getWeatherData(loc.Key);
    let f = await getForecast(loc.Key);

    document.getElementById("result").innerHTML = `
        <h2>${loc.LocalizedName}</h2>
        <img src="${icon(w.WeatherIcon)}">
        <p>${w.WeatherText}</p>
        <p>🌡 ${w.Temperature.Metric.Value} °C</p>
    `;

    let forecastHTML = "";
    f.forEach(day => {
        forecastHTML += `
            <div class="card">
                <p>${new Date(day.Date).toDateString().slice(0,3)}</p>
                <img src="${icon(day.Day.Icon)}">
                <p>${day.Temperature.Maximum.Value}°C</p>
            </div>
        `;
    });

    document.getElementById("forecast").innerHTML = forecastHTML;
}

async function getWeather(){
    let city = document.getElementById("city").value;
    let loc = await getLocation(city);
    if(loc) showWeather(loc);
    else alert("City not found");
}

function getLocationWeather(){
    navigator.geolocation.getCurrentPosition(async pos=>{
        let lat = pos.coords.latitude;
        let lon = pos.coords.longitude;

        let res = await fetch(`https://dataservice.accuweather.com/locations/v1/cities/geoposition/search?apikey=${API_KEY}&q=${lat},${lon}`);
        let data = await res.json();

        showWeather(data);
    });
}

function toggleDark(){
    document.body.classList.toggle("dark");
}
</script>

</body>
</html>
