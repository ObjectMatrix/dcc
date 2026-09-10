<img width="1408" height="768" alt="weather" src="https://github.com/user-attachments/assets/cd8fff7d-df60-40d7-8c41-7470fd3b372d" />

<hr /> 

<h1> From Sky to Screen: How We magic Up Your Daily Weather Forecast</h1>

<img width="896" height="1195" alt="model" src="https://github.com/user-attachments/assets/46171fe6-b56c-48a8-80af-a97f3f27769a" />

```


import requests



def validate_zip(zip_code):
    return zip_code.isdigit() and len(zip_code) == 5


def get_condition(weather_code):
    if weather_code == 0:
        return "Sunny"
    if weather_code in [1, 2, 3]:
        return "Cloudy"
    if weather_code in [71, 73, 75]:
        return "Snowy"
    if weather_code >= 95:
        return "Thunderstorm"
    return "Rainy"



def get_weather(zip_code):
    if not validate_zip(zip_code):
        return {"error": "That doesn't look like a valid ZIP code."}

    try:
        zip_data = requests.get(
            f"http://api.zippopotam.us/us/{zip_code}", timeout=10
        )
        if zip_data.status_code == 404:
            return {"error": f"ZIP code {zip_code} was not found."}
        place = zip_data.json()["places"][0]
        print(place)
        weather_data = requests.get(
            "http://api.open-meteo.com/v1/forecast",
            params={
                "latitude": place["latitude"],
                "longitude": place["longitude"],
                "current": "temperature_2m,weather_code,snowfall",
                "temperature_unit": "fahrenheit",
            },
            timeout=10,
        ).json()["current"]

        return {
            "city": place["place name"],
            "condition": get_condition(weather_data["weather_code"]),
            "temp": round(weather_data["temperature_2m"]),
            "snowfall": weather_data["snowfall"],
        }
    except requests.RequestException:
        return {"error": "Could not get weather data. Please try again."}


def get_advice(condition):
    if condition == "Rainy":
        return "Bring an umbrella."
    if condition == "Sunny":
        return "Wear sunscreen."
    if condition == "Snowy":
        return "Bundle up, it's cold out there."
    if condition == "Thunderstorm":
        return "Stay indoors if you can."
    return "Enjoy your day!"



def display_weather(zip_code, weather):
    if "error" in weather:
        print(weather["error"])
        return

    advice = get_advice(weather["condition"])
    print(f"Weather for {weather['city']} ({zip_code}):")
    print(f"  Condition: {weather['condition']}")
    print(f"  Temperature: {weather['temp']} F")
    print(f"  Advice: {advice}")



def show_weather(zip_code):
    weather = get_weather(zip_code)
    display_weather(zip_code, weather)


```
