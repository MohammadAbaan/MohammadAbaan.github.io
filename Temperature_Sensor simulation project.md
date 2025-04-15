# Abaan_TemperatureSensor
Assignment 3 part 2 work

## Overview
This project simulates a temperature sensor system, which generates temperature readings and performs anomaly detection. It includes features such as storing sensor data, clearing history, detecting anomalies, and logging data. The project is built using C# and .NET. Branches were made using gitbash commands in vs code and the coding work and testing was all done in visual studio before merging and pushing to GitHub.

## Setup Instructions and Running the program

Navigate to the project directory, this is my path:
```
cd "C:\Users\Abaan\OneDrive\Desktop\Assignment3_Abaan\TemperatureSensor"
dotnet run
```

Navigate to the Tests folder, this is my path:
```
cd "C:\Users\Abaan\OneDrive\Desktop\Assignment3_Abaan\TemperatureSensor\Tests"
dotnet test
```

This will execute all the test cases in the Tests project, and the results will be displayed in the terminal.

## Methods and Functions

**StoreData(double temperature):**
Description: Store data both in memory and in the database
Usage:
```
public static void StoreData(double temperature)
        {
            var reading = new SensorData(temperature);

            // Add to in-memory history
            History.Add(reading);
            Console.WriteLine($"Added to memory: {reading.Timestamp} - {reading.Temperature}°C");

            // Add to the database
            string dbPath = "SensorData.db";
            using (var connection = new SQLiteConnection($"Data Source={dbPath};Version=3;"))
            {
                connection.Open();
                string insertQuery = @"
                    INSERT INTO SensorReadings (Temperature, Timestamp)
                    VALUES (@temperature, @timestamp)";

                using (var command = new SQLiteCommand(insertQuery, connection))
                {
                    command.Parameters.AddWithValue("@temperature", reading.Temperature);
                    command.Parameters.AddWithValue("@timestamp", reading.Timestamp.ToString("o"));
                    command.ExecuteNonQuery();
                }

                Console.WriteLine("Stored data in database.");
            }
        }
```
This will store a reading of the temperature in the database and in the History list and output the timestamped reading to the console too.


**ClearHistory():**
Description: Clears all stored sensor readings from the History list.
Usage Example:
```
History.Clear();
```
This will remove all entries from the history, effectively "resetting" the sensor data storage.

**GetAverageTemperature():**
Description: Calculates and returns the average of all stored temperature readings in the history.
Usage Example:
```
return History.Count > 0 ? History.Average(h => h.Temperature) : 0;
```
If there are readings of 22.0°C, 24.0°C, and 23.0°C, the method will return 23.0°C.

**PrintHistory():**
Description: Prints out all stored sensor readings, including the timestamp and temperature, to the console.
Usage Example:
```
if (!History.Any())
            {
                Console.WriteLine("No data history available.");
                return;
            }

            Console.WriteLine("Data History:");
            foreach (var data in History)
            {
                Console.WriteLine($"Timestamp: {data.Timestamp}, Temperature: {data.Temperature}°C");
            }
```
This will print each sensor reading stored in the History list, along with its timestamp, to the console.

**ShutdownSensor():**
Description: Shuts down the sensor by clearing the stored data history and printing a shutdown message.
Usage Example:
```
SensorData.ClearHistory();  // Clear the stored data history
Console.WriteLine("Sensor has been successfully shut down and history cleared.");
```
This will clear all stored data and output a message indicating the sensor has been shut down.

**SmoothData(int period):**
Description: Calculates the average temperature of the last n sensor readings. The n is provided by the period argument.
Usage Example:
```
double smoothedTemperature = SensorData.SmoothData(5);  // Average of last 5 readings
Console.WriteLine($"Smoothed Temperature: {smoothedTemperature}°C");
```
If the last 5 readings are: 22.0, 23.0, 24.0, 22.5, 23.5, the method will return 23.0°C.

**DetectAnomaly(double sensorData):**
Description: Detects if a given temperature reading is an anomaly. An anomaly is defined as a reading that is more than 0.5°C outside the range of the minimum and maximum temperature readings stored in the history.
Usage Example:
```
bool anomalyDetected = SensorData.DetectAnomaly(sensorData);// Pass sensor data as argument
Console.WriteLine($"Anomaly Detected: {anomalyDetected}");
```
If the History contains temperatures like 22.0°C and 24.0°C, the method will return true for a reading of 30.0°C (since it's more than 0.5°C above the maximum stored value of 24.0°C).


## Testing Procedures
The testing procedures for the TemperatureSensor application are implemented using xUnit. The tests are organized in the SensorTests class, which ensures that the functionality of the SensorData class is validated. The tests focus on various aspects, such as storing data, clearing history, calculating averages, detecting anomalies, and ensuring that the methods behave as expected.

**Test Setup**
Before each test is run, the SensorData.ClearHistory() method is called in the constructor to clear any existing sensor data. This ensures that each test runs in isolation and does not rely on data from previous tests.
```
public class SensorTests
    {
        // Constructor for the test class to initialize the database and clear any previous data
        public SensorTests()
        {
            // Initialize the database (ensure the table exists)
            SensorData.InitializeDatabase();

            // Clear history and database before each test to ensure isolated test cases
            SensorData.ClearHistory();
            SensorData.ClearDatabase();
        }
)
```
This prevents tests from failing due to unintended side effects caused by shared state across tests.

**StoreData_ShouldAddDataToHistory**

Description: Verifies that when new sensor data is stored, it is added to the history.
Steps:
Store a valid sensor reading (23.5°C).
Assert that the History list contains one item and that the stored temperature matches the expected value.
Purpose: Ensures that the StoreData method correctly adds data to the history.
Example:
```
Assert.Single(SensorData.History); // Validate in-memory storage
Assert.Equal(23.5, SensorData.History[0].Temperature);

// Validate database storage
var rowCount = SensorData.GetDatabaseRowCount();
Assert.Equal(1, rowCount);

```

**ClearHistory_ShouldRemoveAllData**

Description: Verifies that the ClearHistory method removes all stored data.
Steps:
Store two sensor readings (23.5°C and 24.0°C).
Clear the history.
Assert that the History list is empty.
Purpose: Confirms that the ClearHistory method behaves as expected.
Example:
```
Assert.Empty(SensorData.History);
```

**GetAverageTemperature_ShouldReturnCorrectAverage**

Description: Verifies that the method calculates the correct average temperature.
Steps:
Store three readings (22.0°C, 24.0°C, 23.0°C).
Calculate the average temperature using GetAverageTemperature.
Assert that the calculated average matches the expected value (23.0°C).
Purpose: Ensures that the method calculates the average temperature accurately.
Example:
```
Assert.Equal(23.0, average);
```

**PrintHistory_ShouldOutputData**

Description: Verifies that the PrintHistory method correctly outputs all stored sensor data.
Steps:
Store two readings (23.0°C and 22.5°C).
Call the PrintHistory method.
Assert that the history is printed to the console.
Purpose: Ensures that the PrintHistory method prints the history to the console as expected.
Example:
```
Assert.Equal(2, SensorData.History.Count); // Ensure data is stored in memory
```

**DetectAnomaly_ShouldDetectOutOfRangeTemperature**

Description: Verifies that the anomaly detection function correctly identifies temperatures outside the expected range.
Steps:
Store a valid sensor reading (23.5°C).
Check for anomalies using values both within (23.5°C), below (21.5°C), and above (25.0°C) the valid range.
Assert that anomalies are detected for out-of-range values (21.5°C and 25.0°C), but not for the valid reading (23.5°C).
Purpose: Ensures that the DetectAnomaly function works correctly for valid and invalid data.
Example:
```
Assert.False(SensorData.DetectAnomaly(23.5)); // Valid data is not an anomaly
Assert.True(SensorData.DetectAnomaly(21.5)); // Low data triggers anomaly
Assert.True(SensorData.DetectAnomaly(25.5)); // High data triggers anomaly
```

**SimulateData_ShouldReturnValidSimulatedData**

Description: Verifies that the SimulateData method returns a valid simulated temperature within the expected range.
Steps:
Simulate data for a sensor with a valid range (22.0°C to 24.0°C).
Assert that the simulated data is within the range.
Purpose: Ensures that the data simulation works correctly within the specified bounds.
Example:
```
Assert.InRange(simulatedData, 22.0, 24.0); // Simulated data must be within range
```
