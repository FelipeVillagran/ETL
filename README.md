# ETL
Extracción, transformación y carga de datos meteorológicos.

## rx_poc.sh

```bash
#!/bin/bash

# Asignar el nombre de la ciudad a Casablanca
City=Casablanca

# Obtener la temperatura actual y guardarla en la variable obs_temp
obs_temp=$(curl -s wttr.in/$city?T | grep -m 1 '°.' | grep -Eo -e '-?[[:digit:]].*')
echo "The current Temperature of $city: $obs_temp"

# Obtener la temperatura pronosticada para el mediodía de mañana y guardarla en la variable fc_temp
fc_temp=$(curl -s wttr.in/$city?T | head -23 | tail -1 | grep '°.' | cut -d 'C' -f2 | grep -Eo -e '-?[[:digit:]].*')
echo "The forecasted temperature for noon tomorrow for $city: $fc_temp C"

# Almacenar la hora actual, día, mes y año en variables correspondientes
hour=$(TZ='Africa/Casablanca' date -u +%H) 
day=$(TZ='Africa/Casablanca' date -u +%d) 
month=$(TZ='Africa/Casablanca' date +%m)
year=$(TZ='Africa/Casablanca' date +%Y)

# Fusionar los campos en un registro tabulado
record=$(echo -e "$year\t$month\t$day\t$obs_temp\t$fc_temp C")

# Agregar el registro al archivo de registro meteorológico
echo $record >> rx_poc.log

# Editar el archivo crontab:
crontab -e

0 8 * * * /home/project/rx_poc.sh

#fc_accuracy.sh
#!/bin/bash

# Crear el archivo de precisión histórica del pronóstico con encabezado
echo -e "year\tmonth\tday\tobs_temp\tfc_temp\taccuracy\taccuracy_range" > historical_fc_accuracy.tsv

# Iterar a través del archivo de registro meteorológico
while read -r line1 && read -r line2; do
    # Extraer las temperaturas pronosticadas y observadas para hoy y ayer
    yesterday_fc=$(echo "$line1" | cut -f5 | sed 's/[^0-9.-]//g')
    today_temp=$(echo "$line2" | cut -f4 | sed 's/[^0-9.-]//g')

    # Verificar si las temperaturas son numéricas
    if [[ ! "$yesterday_fc" =~ ^[0-9.-]+$ ]] || [[ ! "$today_temp" =~ ^[0-9.-]+$ ]]; then
        # Saltar la línea si las temperaturas no son numéricas
        continue
    fi

    # Calcular la precisión del pronóstico
    accuracy=$((yesterday_fc - today_temp))

    # Asignar una etiqueta de precisión según el rango
    if [ -1 -le $accuracy ] && [ $accuracy -le 1 ]; then
        accuracy_range=excellent
    elif [ -2 -le $accuracy ] && [ $accuracy -le 2 ]; then
        accuracy_range=good
    elif [ -3 -le $accuracy ] && [ $accuracy -le 3 ]; then
        accuracy_range=fair
    else
        accuracy_range=poor
    fi

    # Extraer información de la fecha
    year=$(echo "$line2" | cut -f1)
    month=$(echo "$line2" | cut -f2)
    day=$(echo "$line2" | cut -f3)

    # Agregar el registro al archivo de precisión histórica del pronóstico
    echo -e "$year\t$month\t$day\t$today_temp\t$yesterday_fc\t$accuracy\t$accuracy_range" >> historical_fc_accuracy.tsv
done < rx_poc.log


#weekly_stats.sh

#!/bin/bash

# Cargar las precisiones históricas en un array que cubra la última semana de datos
echo $(tail -7 synthetic_historical_fc_accuracy.tsv | cut -f6) > scratch.txt
week_fc=($(echo $(cat scratch.txt)))

# Validar el resultado
for i in {0..6}; do
    echo ${week_fc[$i]}
done

# Convertir valores negativos a positivos
for i in {0..6}; do
   



