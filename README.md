# mansi_malik_AssignmentTimecard
import pandas as pd
from datetime import datetime, timedelta

# Define the file path (adjust this to your file's location)
file_path = "C:/Users/shiva/Downloads/Assignment_Timecard.csv"
# Load the CSV file into a DataFrame with automatic header inference
df = pd.read_csv(file_path)

# Helper function to calculate the time difference in hours
def calculate_time_difference(start_time, end_time):
    try:
        start_time = datetime.strptime(start_time, '%m/%d/%Y %I:%M %p')
        end_time = datetime.strptime(end_time, '%m/%d/%Y %I:%M %p')
    except ValueError:
        print(f"Skipping row with invalid date or time format - start_time: {start_time}, end_time: {end_time}")
        return None
    
    time_difference = (end_time - start_time).total_seconds() / 3600  # Convert to hours
    return time_difference

# Initialize variables for consecutive day tracking
consecutive_days_count = 0
prev_date = None
prev_name = None

# Iterate through the DataFrame
for index, row in df.iterrows():
    name = row['Employee Name']
    position = row['Position ID']
    date = row['Pay Cycle End Date']
    start_time = row['Time']
    end_time = row['Time Out']

    # Check if any required field is empty
    if any(pd.isna([name, position, date, start_time, end_time])):
        print(f"Skipping row with missing or incomplete data: {row}")
        continue

    try:
        current_date = datetime.strptime(date, '%m/%d/%Y')
    except ValueError:
        print(f"Skipping row with invalid date format: {date}")
        continue

    # Check for employees who worked for 7 consecutive days
    if prev_name == name:
        if (current_date - prev_date).days == 1:
            consecutive_days_count += 1
        else:
            consecutive_days_count = 1
    else:
        consecutive_days_count = 1

    if consecutive_days_count == 7:
        print(f"{name} ({position}) worked for 7 consecutive days.")

    # Check for employees with valid time data
    time_difference = calculate_time_difference(start_time, end_time)
    if time_difference is None:
        continue  # Skip rows with invalid time data

    # Check for employees with less than 10 hours between shifts but greater than 1 hour
    if 1 < time_difference < 10:
        print(f"{name} ({position}) has less than 10 hours between shifts but greater than 1 hour.")

    # Check for employees who worked for more than 14 hours in a single shift
    if time_difference > 14:
        print(f"{name} ({position}) worked for more than 14 hours in a single shift.")

    prev_date = current_date
    prev_name = name
