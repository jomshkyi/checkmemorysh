#!/bin/bash

#get the memory usage using awk
MEMORY_USAGE=$(free | grep Mem | awk '{printf("%.0f"), $3/$2 * 100.0}')


#function for help to inform the usage regarding the usage of script
function showhelp {
   echo "Usage: checkscript -w [warning treshold in %] -c [critical treshold in %] -e [email address]"
   echo "Example:    checkscript -w 60 -c 90 -e email@mine.com"
   echo "The options were not positional and can be used in any order"
   echo "The options are mandatory -w -c -e and parameter is required"
   echo " The parameters of -c must be greater than -w"
   echo " example: -w 60 -c 90"
}


# Show help when no argument is provided.
if [ $# -eq 0 ]; then   #ensure that there is argument
    echo "No argument provided";
    showhelp
    exit 0
elif [ $# -eq 6 ]; then   #ensure that the 3 options and 3 argument are complete
   echo "The option and parameters were complete";
else
   echo "The option and parameters were incomplete"
    showhelp
    exit 0
fi
 
while getopts ":w:c:e:" opt; do
  case $opt in
     w)
      warn=$OPTARG
        ;;
    c)
      crit=$OPTARG
	;;
    e)
      mail=$OPTARG
	;;
    \?)
      echo "Invalid option: -$OPTARG" >&2
      exit 0
      ;;
    :)     #when no parameter set
     #echo "Option -$OPTARG requires an argument." >&2
     echo "Incomplete parameter. Please provide the required parameter."
      exit 0
      ;;
  esac
done


if [ $warn -lt $crit ]; then   #script ensure that warning treshold was less than crit treshold
   	if [ $MEMORY_USAGE -lt $warn ]; then
   		echo "The memory usage is normarl state: 0"
		exit 0
	elif [[ $MEMORY_USAGE -ge $warn && $MEMORY_USAGE -lt $crit ]]; then
   		echo "The mermory usage is in warning state: 1";
		exit 1
	elif [ $MEMORY_USAGE -ge $crit ]; then
   		echo "The memory usage is in critical state. Please free up some memory space: 2";
		$(top -b -a | head -n 17 | tail | awk '{ print $1" "$10" "$12 }'  > /tmp/top_process.txt)
		datetime=$(date +%Y%m%d" "%H":"%M)
		$(mail -s "$datetime memory check - critical" < /tmp/top_process.txt $mail)
		echo "$datetime"
		exit 2
	fi

else
   echo "The warning treshold must be less than the critical treshold";
   echo "The value must be greater that $warn"
   exit 0
fi
#exit 1
