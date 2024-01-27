# ETL
Extracción, transformación y carga de datos meteorología. 

#!/bin/bash

# Asignar el nombre de la ciudad a Casablanca
City=Casablanca

#!/bin/bash

# Assign the city name to Casablanca
city=Casablanca

# Extract the current temperature and save it in the obs_temp variable
obs_temp=$(curl -s wttr.in/$city?T | grep -m 1 '°.' | grep -Eo -e '-?[[:digit:]].*')
echo "The current Temperature of $city: $obs_temp"

# Extract the forecast temperature for noon tomorrow and save it in fc_temp variable
fc_temp=$(curl -s wttr.in/$city?T | head -23 | tail -1 | grep '°.' | cut -d 'C' -f2 | grep -Eo -e '-?[[:digit:]].*')
echo "The forecasted temperature for noon tomorrow for $city : $fc_temp C"

# Store the current hour, day, month, and year in corresponding variables
hour=$(TZ='Africa/Casablanca' date -u +%H) 
day=$(TZ='Africa/Casablanca' date -u +%d) 
month=$(TZ='Africa/Casablanca' date +%m)
year=$(TZ='Africa/Casablanca' date +%Y)

# Merge the fields into a tab-delimited record
record=$(echo -e "$year\t$month\t$day\t$obs_temp\t$fc_temp C")

# Append the record to the weather log file
echo $record >> rx_poc.log

Edite el archivo crontab:

crontab -e

Agregue la siguiente línea al final del archivo:

0 8 * * * /home/project/rx_poc.sh

#! /bin/bash
 
#Assign city name as Casablanca
city=Casablanca

#Obtain the weather report for Casablanca
curl -s wttr.in/$city?T --output weather_report

#To extract Current Temperature
obs_temp=$(curl -s wttr.in/$city?T | grep -m 1 '°.' | grep -Eo -e '-?[[:digit:]].*')
echo "The current Temperature of $city: $obs_temp"

# To extract the forecast tempearature for noon tomorrow
fc_temp=$(curl -s wttr.in/$city?T | head -23 | tail -1 | grep '°.' | cut -d 'C' -f2 | grep -Eo -e '-?[[:digit:]].*')
echo "The forecasted temperature for noon tomorrow for $city : $fc_temp C"

#Assign Country and City to variable TZ
TZ='Morocco/Casablanca'


# Use command substitution to store the current day, month, and year in corresponding shell variables:
hour=$(TZ='Morocco/Casablanca' date -u +%H)
day=$(TZ='Morocco/Casablanca' date -u +%d)
month=$(TZ='Morocco/Casablanca' date +%m)
year=$(TZ='Morocco/Casablanca' date +%Y)


# Log the weather
record=$(echo -e "$year\t$month\t$day\t$obs_temp\t$fc_temp C")
echo $record>>rx_poc.log


#!/bin/bash

# Create the historical forecast accuracy file with header
echo -e "year\tmonth\tday\tobs_temp\tfc_temp\taccuracy\taccuracy_range" > historical_fc_accuracy.tsv

# Iterate through the weather log file
while read -r line1 && read -r line2; do
    # Extract forecasted and observed temperatures for today and yesterday
    yesterday_fc=$(echo "$line1" | cut -f5 | sed 's/[^0-9.-]//g')
    today_temp=$(echo "$line2" | cut -f4 | sed 's/[^0-9.-]//g')

    # Check if temperatures are numeric
    if [[ ! "$yesterday_fc" =~ ^[0-9.-]+$ ]] || [[ ! "$today_temp" =~ ^[0-9.-]+$ ]]; then
        # Skip the line if temperatures are not numeric
        continue
    fi

    # Calculate forecast accuracy
    accuracy=$((yesterday_fc - today_temp))

    # Assign accuracy label based on the range
    if [ -1 -le $accuracy ] && [ $accuracy -le 1 ]; then
        accuracy_range=excellent
    elif [ -2 -le $accuracy ] && [ $accuracy -le 2 ]; then
        accuracy_range=good
    elif [ -3 -le $accuracy ] && [ $accuracy -le 3 ]; then
        accuracy_range=fair
    else
        accuracy_range=poor
    fi

    # Extract date information
    year=$(echo "$line2" | cut -f1)
    month=$(echo "$line2" | cut -f2)
    day=$(echo "$line2" | cut -f3)

    # Append the record to historical forecast accuracy file
    echo -e "$year\t$month\t$day\t$today_temp\t$yesterday_fc\t$accuracy\t$accuracy_range" >> historical_fc_accuracy.tsv

done < rx_poc.log


Este script generalizado procesa todos los días en el archivo de registro meteorológico y agrega la precisión del pronóstico histórico al archivo.

# Initialize variables for minimum and maximum
minimum=${week_fc[0]}
maximum=${week_fc[0]}

# Find the minimum and maximum absolute errors
for item in "${week_fc[@]}"; do
    # Use absolute value function to ensure positive values
    abs_item=$(echo $item | awk '{print ($1 < 0) ? -$1 : $1}')

    echo "item: $item, abs_item: $abs_item, minimum: $minimum, maximum: $maximum"

    if [[ $minimum -gt $abs_item ]]; then
        minimum=$abs_item
    fi
    if [[ $maximum -lt $abs_item ]]; then
        maximum=$abs_item
    fi
done

# Display the results
echo "minimum absolute error = $minimum"
echo "maximum absolute error = $maximum"


