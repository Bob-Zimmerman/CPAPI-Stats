# CPAPI-Stats
Data related to Check Point API calls for statistical analysis

This data came from my attempts to optimize my API calls for some development. The VMs are running in VirtualBox on a macpro5,1 with two Xeon X5675 processors, 96 GB of 1066 MHz DDR3 RAM, and a 2 TB Western Digital Blue SATA SSD.

On the VMs, I used this script:
```
#!/usr/bin/env bash
TIMEFORMAT='%R'
filePrefix="vm$(egrep "^processor\s" /proc/cpuinfo | wc -l)$(grep MemTotal /proc/meminfo | awk '{GB = $2/1000000} END {printf "%02.0f\n", GB}')"
limits=(500 400 300 200 100 1)
details=("Full" "UID")
for callLimit in ${limits[@]}; do
	for callDetails in ${details[@]}; do
		{ for iteration in $(seq 1 1000);do
			sleep 2
			time mgmt_cli -r true \
				--format json \
				show application-sites \
				limit "${callLimit}" \
				details-level "${callDetails}" \
				> /dev/null
			done } 2> "${filePrefix}AppSites${callLimit}${callDetails}.txt"
		done
	done
```

On the 2200, I use a similar script, but with a manual file prefix.

For basic analysis, I've been using this:
```
for FILE in $(ls -1 *500Full.txt);do
	echo $FILE
	echo "min = $(sort -n $FILE | head -n 1)"
	awk '{sum += $1} END {print "mean = " sum/NR}' $FILE
	awk '{sum+=$1; sumsq+=$1*$1} END {print "stdev = " sqrt(sumsq/NR - (sum/NR)**2)}' $FILE
	echo "max = $(sort -n $FILE | tail -n 1)"
	echo ""
	done
```
