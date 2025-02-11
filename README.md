mkdir weather-dashboard-demo
cd weather-dashboard-demo
mkdir src tests data
touch src/__init__.py src/weather_dashboard.py
touch requirements.txt README.md .env
git init
git branch -M main
echo ".env" >> .gitignore
echo "__pycache__/" >> .gitignore
echo "*.zip" >> .gitignore
echo "boto3==1.26.137" >> requirements.txt
echo "python-dotenv==1.0.0" .. requirements.txt
echo "request==2.28.2: >> requirements.txt
pip install -r requirements.txt
aws configure
echo "OPENWEATHER_API_KEY=your_api_key_here" >> .env
echo "AWS_BUCKET_NAME=weather-dashboard-${RANDOM}" >> .env
add code to weather_dashboard.py:
```py
import os
import json
import boto3
import requests
from datetime import datetime
from dotenv import load_dotenv

# Load environment variables
load_dotenv()

class WeatherDashboard:
    def __init__(self):
        self.api_key = os.getenv('OPENWEATHER_API_KEY')
        self.bucket_name = os.getenv('AWS_BUCKET_NAME')
        self.s3_client = boto3.client('s3')

    def create_bucket_if_not_exists(self):
        """Create S3 bucket if it doesn't exist"""
        try:
            self.s3_client.head_bucket(Bucket=self.bucket_name)
            print(f"Bucket {self.bucket_name} exists")
        except:
            print(f"Creating bucket {self.bucket_name}")
        try:
            # Simpler creation for us-east-1
            self.s3_client.create_bucket(Bucket=self.bucket_name)
            print(f"Successfully created bucket {self.bucket_name}")
        except Exception as e:
            print(f"Error creating bucket: {e}")

    def fetch_weather(self, city):
        """Fetch weather data from OpenWeather API"""
        base_url = "http://api.openweathermap.org/data/2.5/weather"
        params = {
            "q": city,
            "appid": self.api_key,
            "units": "imperial"
        }
        
        try:
            response = requests.get(base_url, params=params)
            response.raise_for_status()
            return response.json()
        except requests.exceptions.RequestException as e:
            print(f"Error fetching weather data: {e}")
            return None

    def save_to_s3(self, weather_data, city):
        """Save weather data to S3 bucket"""
        if not weather_data:
            return False
            
        timestamp = datetime.now().strftime('%Y%m%d-%H%M%S')
        file_name = f"weather-data/{city}-{timestamp}.json"
        
        try:
            weather_data['timestamp'] = timestamp
            self.s3_client.put_object(
                Bucket=self.bucket_name,
                Key=file_name,
                Body=json.dumps(weather_data),
                ContentType='application/json'
            )
            print(f"Successfully saved data for {city} to S3")
            return True
        except Exception as e:
            print(f"Error saving to S3: {e}")
            return False

def main():
    dashboard = WeatherDashboard()
    
    # Create bucket if needed
    dashboard.create_bucket_if_not_exists()
    
    cities = ["Philadelphia", "Seattle", "New York"]
    
    for city in cities:
        print(f"\nFetching weather for {city}...")
        weather_data = dashboard.fetch_weather(city)
        if weather_data:
            temp = weather_data['main']['temp']
            feels_like = weather_data['main']['feels_like']
            humidity = weather_data['main']['humidity']
            description = weather_data['weather'][0]['description']
            
            print(f"Temperature: {temp}°F")
            print(f"Feels like: {feels_like}°F")
            print(f"Humidity: {humidity}%")
            print(f"Conditions: {description}")
            
            # Save to S3
            success = dashboard.save_to_s3(weather_data, city)
            if success:
                print(f"Weather data for {city} saved to S3!")
        else:
            print(f"Failed to fetch weather data for {city}")

if __name__ == "__main__":
    main()
    ```
python src/weather_dashboard.py

Returns: 

```
Creating bucket weather-dashboard-20818
Successfully created bucket weather-dashboard-20818

Fetching weather for Philadelphia...
Temperature: 35.96°F
Feels like: 30.33°F
Humidity: 46%
Conditions: overcast clouds
Successfully saved data for Philadelphia to S3
Weather data for Philadelphia saved to S3!

Fetching weather for Seattle...
Temperature: 32.36°F
Feels like: 24.58°F
Humidity: 54%
Conditions: clear sky
Successfully saved data for Seattle to S3
Weather data for Seattle saved to S3!

Fetching weather for New York...
Temperature: 34.52°F
Feels like: 27.88°F
Humidity: 47%
Conditions: light rain
Successfully saved data for New York to S3
Weather data for New York saved to S3!
```
Got to AWS S3 (image of weather-dashboard bucket)

Click on bucket (image)
Click on weather-data (image)