#!/bin/bash


RED='\033[0;31m'   
WHITE='\033[1;37m'
#get the memory usage using awk
MEMORY_USAGE=$(free | grep Mem | awk '{printf("%.0f"), $3/$2 * 100.0}')


#function for help to inform the usage regarding the usage of script
function showhelp {
   echo -e "${WHITE}Usage:\n      memory_check -w [warning treshold in %] -c [critical treshold in %] -e [email address]"
   echo -e "Example:\n      memory_check -w 60 -c 90 -e email@mine.com\n"
   echo "The options were not positional and can be used in any order"
   echo "The options are mandatory -w -c -e and parameters are required"
   echo -e "The parameters of -c must be greater than -w \n"
}


# Show help when no argument is provided.
if [ $# -eq 0 ]; then   #ensure that there is argument
    echo -e "\n${RED} No argument provided!\n";
    showhelp
    exit 0
elif [ $# -eq 6 ]; then   #ensure that the 3 options and 3 argument are complete
   echo -e  "\nOptions and Parameters will be accepted!\n";
else
   echo -e "\n${RED} Incomplete option or parameter!\n"
    showhelp
    exit 0
fi
 
while getopts ":w:c:e:" opt; do
  case $opt in
     w)
      warning_treshold=$OPTARG
        ;;
    c)
      critical_treshold=$OPTARG
	;;
    e)
      email_address=$OPTARG
	;;
  esac
done


if [ $warning_treshold -lt $critical_treshold ]; then   #script ensure that warning treshold was less than crit treshold
   	if [ $MEMORY_USAGE -lt $warning_treshold ]; then
   		echo -e "The memory usage is normarl state: 0\n"
		exit 0
	elif [[ $MEMORY_USAGE -ge $warning_treshold && $MEMORY_USAGE -lt $critical_treshold ]]; then
   		echo -e "The mermory usage is in warning state: 1\n";
		exit 1
	elif [ $MEMORY_USAGE -ge $critical_treshold ]; then
   		echo -e "The memory usage is in critical state. Please free up some memory space: 2\n";
		$(top -b -a | head -n 17 | tail | awk '{ print $1" "$10" "$12 }'  > /tmp/top_process.txt)
		datetime=$(date +%Y%m%d" "%H":"%M)
		$(mail -s "$datetime memory check - critical" < /tmp/top_process.txt $email_address)
		echo -e "Mail Sent! Please check the inbox of $email_address for top ten process\n"
		exit 2
	fi

else
   echo -e "${RED}The warning treshold must be less than the critical treshold!\n";
   echo -e "${WHITE}The value of critical treshold must be greater than $warning_treshold.\n Please try again!\n"
   exit 0
fi
#exit 1
