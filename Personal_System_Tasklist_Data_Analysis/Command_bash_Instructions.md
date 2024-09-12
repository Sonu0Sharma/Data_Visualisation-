

# Complete Process Overview: 

## 1. Get the Task List Using the `top` Command:
The `top` command is used to display real-time process data (like task lists) on macOS.

### Basic Command:

```bash
top
```

## 2. Save the Task List to a File Every 10 Seconds for 10 Minutes:
To capture the task list every 10 seconds for 10 minutes and save it to a file, use a Bash script. The data will be appended to the same file at each interval.

### Step-by-Step Guide:

### 1. Open Terminal and Create a Bash Script:
```bash
nano save_top_data.sh
```

### 2. Add the following code to capture the task list and save it every 10 seconds for 10 minutes:

```bash
#!/bin/bash

# Directory to save the top command output
OUTPUT_DIR=~/Desktop/top_data

# Create the directory if it doesn't exist
mkdir -p $OUTPUT_DIR

# Output file path
OUTPUT_FILE="$OUTPUT_DIR/combined_top_data.txt"

# Duration in seconds (10 minutes)
DURATION=600

# Interval between captures in seconds (e.g., 10 seconds)
INTERVAL=10

# Start time
START_TIME=$(date +%s)

# Clear the output file if it exists
> $OUTPUT_FILE

while [ $(($(date +%s) - START_TIME)) -lt $DURATION ]; do
    # Generate a timestamp for each capture
    TIMESTAMP=$(date +"%Y-%m-%d %H:%M:%S")

    # Append the timestamp to the output file
    echo "Timestamp: $TIMESTAMP" >> $OUTPUT_FILE

    # Capture the top command output and append it to the file
    top -l 1 >> $OUTPUT_FILE

    # Add a separator between intervals
    echo -e "\n------------------------------\n" >> $OUTPUT_FILE

    # Wait for the interval
    sleep $INTERVAL
done
```

### 3. Save and exit the script (`CTRL + X`, `Y`, `Enter`).

### 4. Make the script executable:
```bash
chmod +x save_top_data.sh
```

### 5. Run the script:
```bash
./save_top_data.sh
```

This script will run the `top` command every 10 seconds for 10 minutes and save the combined output to `combined_top_data.txt`.

---

## 3. Check the Number of Lines in the Combined Text File:
To see how many lines of data are in the combined text file, use the following command:

```bash
wc -l ~/Desktop/top_data/combined_top_data.txt
```

---

## 4. Convert the Combined Text File to CSV:
Once the `top` data is saved to a text file, you can convert it to a CSV file for easier processing and analysis.

### Step-by-Step Guide:

### 1. Create a new script to convert the text file into a CSV file:

Open a new script file:

```bash
nano convert_to_csv.sh
```

### 2. Add the following code to extract relevant columns (e.g., PID, COMMAND, %CPU, and MEM) and convert the text into CSV:

```bash
#!/bin/bash

# Paths to the input and output files
INPUT_FILE=~/Desktop/top_data/combined_top_data.txt
OUTPUT_FILE=~/Desktop/top_data/top_data.csv

# Write CSV header
echo "Timestamp,PID,COMMAND,%CPU,MEM" > $OUTPUT_FILE

# Temporary variables
timestamp=""
inside_data_section=false

# Process the input file
while IFS= read -r line; do
    # Check for timestamp lines and update the timestamp variable
    if [[ $line =~ Timestamp: ]]; then
        timestamp=$(echo $line | sed 's/Timestamp: //')
        inside_data_section=false
        continue
    fi

    # Identify the start of the data section
    if [[ $line =~ ^PID ]]; then
        inside_data_section=true
        continue
    fi

    # If inside the data section, process the lines
    if $inside_data_section; then
        # Extract relevant columns and write to CSV
        # Adjust the fields based on your `top` output format
        echo "$timestamp,$(echo $line | awk '{print $1","$2","$3","$4}')" >> $OUTPUT_FILE
    fi
done < "$INPUT_FILE"
```

### 3. Save and exit the script (`CTRL + X`, `Y`, `Enter`).

### 4. Make the script executable:
```bash
chmod +x convert_to_csv.sh
```

### 5. Run the script to convert the text file to a CSV:
```bash
./convert_to_csv.sh
```

---

## 5. Result:
After running the script, you will have two important files:

- `combined_top_data.txt`: The raw `top` command data collected over 10 minutes.
- `top_data.csv`: The CSV-formatted version of the data, containing `Timestamp`, `PID`, `COMMAND`, `%CPU`, and `MEM` columns.

---

# Summary of Key Commands:

- **Task List Command:** `top`
- **Saving Task List at Intervals:** Bash script (`save_top_data.sh`)
- **Merging into One File:** Automatically done by appending output in the `save_top_data.sh` script
- **Convert Text File to CSV:** Bash script (`convert_to_csv.sh`)

---

This complete setup will help you capture, save, merge, and convert task list data from macOS `top` into a CSV file for further analysis.

---
