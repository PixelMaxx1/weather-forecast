# weather-forecast
a simple code that helps forecasting weather
import tkinter as tk
from tkinter import messagebox
import requests

# ---------------------------------------------------------------------------
# CORE WEATHER LOGIC
# ---------------------------------------------------------------------------
def get_weather_forecast(city_name):
    """
    Fetches coordinates for a city and uses them to retrieve a 3-day forecast
    from the keyless Open-Meteo API.
    """
    # 1. Geocoding: Turn city name into Latitude and Longitude
    geo_url = f"https://open-meteo.com{city_name}&count=1&language=en&format=json"
    
    try:
        geo_response = requests.get(geo_url).json()
        
        # Check if the city was found
        if not geo_response.get("results"):
            return None, f"City '{city_name}' not found. Please check the spelling."
            
        location_data = geo_response["results"][0]
        lat = location_data["latitude"]
        lon = location_data["longitude"]
        full_name = f"{location_data.get('name')}, {location_data.get('country')}"
        
        # 2. Weather Forecast: Get daily max/min temperatures and weather codes
        forecast_url = f"https://open-meteo.com{lat}&longitude={lon}&daily=weathercode,temperature_2m_max,temperature_2m_min&timezone=auto"
        forecast_response = requests.get(forecast_url).json()
        
        return {
            "location": full_name,
            "daily": forecast_response["daily"]
        }, None
        
    except requests.exceptions.RequestException:
        return None, "Network error. Please check your internet connection."

# ---------------------------------------------------------------------------
# INTERFACE ACTIONS
# ---------------------------------------------------------------------------
def format_weather_code(code):
    """Converts WMO Weather Interpretation Codes into friendly text strings."""
    mapping = {
        0: "Clear sky ☀️", 1: "Mainly clear 🌤️", 2: "Partly cloudy ⛅", 3: "Overcast ☁️",
        45: "Fog 🌫️", 48: "Depositing rime fog 🌫️",
        51: "Light drizzle 🌧️", 53: "Moderate drizzle 🌧️", 55: "Dense drizzle 🌧️",
        61: "Slight rain 🌧️", 63: "Moderate rain 🌧️", 65: "Heavy rain 🌧️",
        71: "Slight snow ❄️", 73: "Moderate snow ❄️", 75: "Heavy snow ❄️",
        95: "Thunderstorm ⛈️"
    }
    return mapping.get(code, "Unspecified Conditions 🤷")

def update_ui():
    """Triggered on button click to handle user input and update layout labels."""
    city = city_entry.get().strip()
    if not city:
        messagebox.showwarning("Warning", "Please enter a city name first.")
        return
        
    # Visual loading feedback
    search_button.config(state="disabled", text="Searching...")
    window.update_idletasks()
    
    data, error = get_weather_forecast(city)
    
    # Re-enable the button
    search_button.config(state="normal", text="Get Forecast")
    
    if error:
        messagebox.showerror("Error", error)
        return
        
    # Render metadata
    location_label.config(text=data["location"])
    
    # Parse arrays out of the API payload
    dates = data["daily"]["time"]
    max_temps = data["daily"]["temperature_2m_max"]
    min_temps = data["daily"]["temperature_2m_min"]
    codes = data["daily"]["weathercode"]
    
    # Update individual day cards (Displaying 3 days of data)
    for i in range(3):
        day_date = dates[i]
        day_condition = format_weather_code(codes[i])
        day_temp = f"Max: {max_temps[i]}°C | Min: {min_temps[i]}°C"
        
        forecast_cards[i]["date"].config(text=day_date)
        forecast_cards[i]["cond"].config(text=day_condition)
        forecast_cards[i]["temp"].config(text=day_temp)

# ---------------------------------------------------------------------------
# APPLICATION LAYOUT (GUI)
# ---------------------------------------------------------------------------
# Create principal application viewport
window = tk.Tk()
window.title("3-Day Weather Forecast")
window.geometry("450x550")
window.configure(bg="#f0f4f8")

# Input Section Header
input_frame = tk.Frame(window, bg="#f0f4f8", pady=20)
input_frame.pack(fill="x")

city_label = tk.Label(input_frame, text="Enter City Name:", font=("Helvetica", 12, "bold"), bg="#f0f4f8", fg="#333333")
city_label.pack(side="left", padx=(30, 10))

city_entry = tk.Entry(input_frame, font=("Helvetica", 12), width=18, bd=2, relief="groove")
city_entry.pack(side="left", padx=5)
city_entry.focus()
# Bind 'Enter' keypress to launch search directly
city_entry.bind("<Return>", lambda event: update_ui())

search_button = tk.Button(input_frame, text="Get Forecast", font=("Helvetica", 10, "bold"), bg="#4a90e2", fg="white", 
                          activebackground="#357abd", activeforeground="white", padx=10, command=update_ui, relief="flat")
search_button.pack(side="left", padx=10)

# Found Location Metadata Container
location_label = tk.Label(window, text="Search for a city to begin", font=("Helvetica", 14, "bold"), bg="#f0f4f8", fg="#4a90e2", pady=10)
location_label.pack()

# Forecast Container Box
forecast_frame = tk.Frame(window, bg="#f0f4f8")
forecast_frame.pack(fill="both", expand=True, padx=30, pady=10)

forecast_cards = []

# Build individual structural rows for the 3-day window display loop
for i in range(3):
    card = tk.LabelFrame(forecast_frame, text=f"Day {i+1}", font=("Helvetica", 10, "italic"), bg="white", bd=1, relief="solid", padx=15, pady=10)
    card.pack(fill="x", pady=8)
    
    lbl_date = tk.Label(card, text="--", font=("Helvetica", 11, "bold"), bg="white", anchor="w")
    lbl_date.pack(fill="x")
    
    lbl_cond = tk.Label(card, text="--", font=("Helvetica", 11), bg="white", fg="#555555", anchor="w")
    lbl_cond.pack(fill="x")
    
    lbl_temp = tk.Label(card, text="--", font=("Helvetica", 11), bg="white", fg="#e67e22", anchor="w")
    lbl_temp.pack(fill="x")
    
    # Store references to widgets for easy programmatic updates
    forecast_cards.append({
        "date": lbl_date,
        "cond": lbl_cond,
        "temp": lbl_temp
    })

# Start application view process loop
window.mainloop()
