<script lang="ts">
    import { onMount } from "svelte";
    import { env } from "$env/dynamic/public";
    import * as echarts from "echarts";

    /**
     * @type {echarts.ECharts}
     */
    let chart: echarts.ECharts;
    let data; 
    let option;
    
  
    let playersList: Player[] = [];

    interface XpAtTime {
        time: string;
        xpForEachSkill: number[];
        totalXp: number;
    };
    
    interface Player {
        name: string;
        xpAtTime: XpAtTime[];
    };

    onMount(async () => {
        await loadSkillData();
        getTimeSeriesDataForPlayer()
        chart = echarts.init(document.getElementById("chart-container"));
        chart.setOption({
            xAxis: {
                // data: ["A", "B", "C", "D", "E", "F"],
                data: getListOfTimesFromData(getTimeSeriesDataForPlayer().xpAtTime),
                boundaryGap: false
            },
            yAxis: {
                min: getMinYAxis(),
                splitLine: {
                    show: false // Set to false to hide the horizontal grid lines
                }
            },
            series: playersList.map((player, index) => ({
                data: getListOfXpDataFromData(player.xpAtTime),
                type: "line",
                smooth: true,
                name: player.name
            })),
            legend: {
                data: playersList.map((player) => player.name),
                // change the text color
                textStyle: {
                    // greyish blue
                    color: 'rgb(248 250 252)'
                }
            },
            // series: [
            //     {
            //         name: "Dummy Data",
            //         type: "line",
            //         smooth: true,
            //         data: getListOfXpDataFromData(getTimeSeriesDataForPlayer().xpAtTime),
            //     },
            // ],
            grid: {
                show: false
            },
            animation: false,
            darkMode: true,
            // change the text color
            textStyle: {
                // greyish blue
                color: 'rgb(248 250 252)'
            },

            // lineStyle: {
            //     color: 'rgba(0, 0, 0, 0.3)', // Adjust the color and transparency here
            //     width: 1 // Adjust the line width if needed
            // }
        });
    });

    function getMinYAxis(): number {
        let minYAxis = 9999999999999;
        for (let player of playersList) {
            for (let xp of player.xpAtTime) {
                if (xp.totalXp < minYAxis) {
                    minYAxis = xp.totalXp
                }
            }
        }

        if (minYAxis > 1000000) {
            // return the floored value to nearest million
            return Math.floor(minYAxis / 1000000) * 1000000
        }
        return 0;
    }

    function getMaxYAxis(): number {
        let maxYAxis = 0;
        for (let player of playersList) {
            for (let xp of player.xpAtTime) {
                if (xp.totalXp > maxYAxis) {
                    maxYAxis = xp.totalXp
                }
            }
        }
        return maxYAxis
    }

    
    function getTimeSeriesDataForPlayer(): Player {
        let playerName = "BB Bright"
        // find the player in the data
        // get the time series data for the player
        // return the time series data

        // code goes here:

        for (let player of playersList) {
            if (player.name == playerName) {
                console.log(player)
                return player
            }
        }

        // throw error
        throw new Error("Couldn't find player");
    }

    function getListOfXpDataFromData(data: XpAtTime[]): number[] {
        let xpData: number[] = [];
        for (let p of data) {
            xpData.push(p.totalXp);
        }
        return xpData;
    }

    function getListOfTimesFromData(data: XpAtTime[]): string[] {
        let times: string[] = [];
        for (let p of data) {
            times.push(p.time);
        }
        return times;
    }

    async function loadSkillData() {
        // json return from this endpoint matches sampleGraphData.json
        const res = await fetch(`${env.PUBLIC_BACKEND_BASE_URL}/getIronXpYear`);
        data = await res.json();
        console.log(data)

        // map the data to the echart line graph
        // each line on the graph represents a player
        // each point on the line represents total xp for all skills
        // to get total xp for all skills sum the xp for each skill (data)
        // graph the xp over time for each player

        // code goes here:
        // for each player

        let mostTimePoints = 0;
        let indexOfMostTimePoints = 0;

        for (let p of data) {
            let player: Player = {
                name: p['name'],
                xpAtTime: []
            }
            
            // for each time point
            for (let t of p['skill_data']) {
                let skillData: XpAtTime = {
                    time: t['time'],
                    xpForEachSkill: [],
                    totalXp: 0
                }
                // for each skill
                for (let s of t['data']) {
                    skillData.xpForEachSkill.push(Number(s));
                    skillData.totalXp += Number(s); // convert string to number for additions;
                }
                player.xpAtTime.push(skillData);
            }
            // reorder player.xpAtTime by time
            player.xpAtTime.sort((a, b) => new Date(a.time).getTime() - new Date(b.time).getTime())
            
            if (player.xpAtTime.length > mostTimePoints) {
                mostTimePoints = player.xpAtTime.length
                indexOfMostTimePoints = playersList.length
            }
            
            playersList.push(player);
        }

        // chaos test
        // for (let player of playersList) {
        //     if (player.name === "BB Bright") {
        //         // remove players 1st data point
        //         player.xpAtTime.shift();
        //     }
        // }

        // some players are missing time data points
        // pad the players if they are missing time points either to the left or to the right (wherever they are missing data)
        // if they are missing data add a new data point at the correct time based on player with most time points
        // pad their xp by using their closest data point

        let playerWithMostData: Player = playersList[indexOfMostTimePoints];

        for (let player of playersList) {
            if (player.xpAtTime.length == mostTimePoints) {
                    continue;
            }
            console.log("This player needs to be appended", player)
            let timePointsToAppend: XpAtTime[] = [];
            for (let time of playerWithMostData.xpAtTime) {
                if (!playerHasDataForThisTime(player, time.time)) {
                    console.log("This player doesn't have this time", player, time.time)
                    timePointsToAppend.push(findPlayersClosestTimePointToCopy(player, time.time))
                }
            }
            console.log("time points to append", player, timePointsToAppend)
            player.xpAtTime = player.xpAtTime.concat(timePointsToAppend);
            player.xpAtTime.sort((a, b) => new Date(a.time).getTime() - new Date(b.time).getTime())
            console.log("Player after appending", player)
        }

    }

    function playerHasDataForThisTime(player: Player, time: string): boolean {
        for (let xp of player.xpAtTime) {
            if (xp.time == time) {
                return true;
            }
        }
        return false;
    }

    function findPlayersClosestTimePointToCopy(player: Player, time: string): XpAtTime {
        // pick the players time that is closest to the input time

        // loop through player.xpAtTime backwards

        for (let i = player.xpAtTime.length - 1; i >= 0; i--) {
            if (player.xpAtTime[i].time < time) {
                return {time: time, xpForEachSkill: player.xpAtTime[i].xpForEachSkill, totalXp: player.xpAtTime[i].totalXp}
            }
        }

        // if got here didn't find a previous time for the player
        return {time: time, xpForEachSkill: [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0], totalXp: 0}


    }
</script>


<div id="chart-container" class="w-half h-4/6 mx-auto my-4 p-4" style="width: 2000px; height : 800px"></div>
