<script lang="ts">
    import { onMount } from "svelte";

    import { page } from '$app/stores'; 
    import { env } from "$env/dynamic/public";

    let playerName = $page.params.playerName

    interface Skill {
        Name: string;
        Level: number;
        Xp: number;
        ProgressPercent: number;
    }

    interface Player {
        Name: string;
        SkillsMap: Map<string, Skill>; 
        Skills: { [key: string]: Skill };
    }

    const skillsMapping: string[] = [
        "Attack", "Hitpoints", "Mining", "Strength", "Agility",
        "Smithing", "Defence", "Herblore", "Fishing", "Ranged", 
        "Thieving", "Cooking", "Prayer", "Crafting",
        "Firemaking", "Magic", "Fletching", "Woodcutting", "Runecraft", "Slayer", 
        "Farming", "Construction", "Hunter"
    ];

    let player: Player;

    let totalLevel = 0;
    
    const skillIcons = new Map<string, string>([
        ["Attack", "/ui/197-0.png"],
        ["Strength", "/ui/198-0.png"],
        ["Defence", "/ui/199-0.png"],
        ["Ranged", "/ui/200-0.png"],
        ["Prayer", "/ui/201-0.png"],
        ["Magic", "/ui/202-0.png"],
        ["Hitpoints", "/ui/203-0.png"],
        ["Agility", "/ui/204-0.png"],
        ["Herblore", "/ui/205-0.png"],
        ["Thieving", "/ui/206-0.png"],
        ["Crafting", "/ui/207-0.png"],
        ["Fletching", "/ui/208-0.png"],
        ["Mining", "/ui/209-0.png"],
        ["Smithing", "/ui/210-0.png"],
        ["Fishing", "/ui/211-0.png"],
        ["Cooking", "/ui/212-0.png"],
        ["Firemaking", "/ui/213-0.png"],
        ["Woodcutting", "/ui/214-0.png"],
        ["Runecraft", "/ui/215-0.png"],
        ["Slayer", "/ui/216-0.png"],
        ["Farming", "/ui/217-0.png"],
        ["Hunter", "/ui/220-0.png"],
        ["Construction", "/ui/221-0.png"]
    ]);

    function getBackgroundColorByPercentage(percentage: number): string {
        const clampedPercentage = Math.max(0, Math.min(percentage, 100));

        let color: number[] = []; // Array to store the RGB color

        switch (true) {
            case clampedPercentage < 20:
                color = [255, 2, 0]; // Red
                break;
            case clampedPercentage < 40:
                color = [223, 124, 12]; // Orange
                break;
            case clampedPercentage < 60:
                color = [223, 245, 24]; // Yellow
                break;
            case clampedPercentage < 80: 
                color = [89, 215, 77]; // Light green
                break;
            default:
                color = [20, 164, 7]; // Green
        }

        return `rgb(${color[0]}, ${color[1]}, ${color[2]})`;
    }

    function computeStyleProgressBar(skillName: string) {
        const percent = player.SkillsMap.get(skillName)?.ProgressPercent;
        if (percent == null) {
            return "";
        }
        return "transform: scaleX(" + (percent / 100) + "); background: " + getBackgroundColorByPercentage(percent);
    }

    async function loadSkillData() {
        const res = await fetch(`${env.PUBLIC_BACKEND_BASE_URL}/getIronData/${playerName}`);
        player = await res.json()
        console.log(player)

        player.SkillsMap = new Map();
        for (const skillName in player.Skills) {
            player.SkillsMap.set(skillName, player.Skills[skillName]);
            totalLevel += player.Skills[skillName].Level;
        }

    }

    onMount(async () => {
        loadSkillData()
    });

</script>


{#if player != null}
    <div id="parent-box">
        <div class="skills-grid">
            {#each skillsMapping as skill}
                <div class="skill">
                    <div class="skill-left">
                        <img alt="skill icon" class="skill-box__icon" src="{skillIcons.get(skill)}" />
                    </div>
                    <div class="skill-right">
                        <div class="current-level">{player.SkillsMap.get(skill)?.Level}</div>
                        <div class="base-level">{player.SkillsMap.get(skill)?.Level}</div>
                    </div>
                    <div class="skill-bottom">
                        <div class="progress-bar" style={computeStyleProgressBar(skill)}></div>
                    </div>
                </div>
            {/each}

            <div class="total-level">
                <img src="/ui/183-0.png" alt="total level img"/>
                <img src="/ui/184-0.png" alt="total level img"/>

                <div class="total-level-inner">
                    <span>Total level:</span>
                    <span id="total-level-number">{totalLevel}</span>
                </div>
            </div>

        </div>
    </div>
{/if}



<style>

    .total-level {
        position: relative;
        text-align: center;
        display: flex;
        justify-content: center;
        align-items: center;
    }

    .total-level-inner {
        position: absolute;
        text-align: center;
        display: flex;
        font-size: 13px;
        flex-direction: column;
        justify-content: center;
        align-items: center;
        line-height: 1em;
    }
    
    .skills-grid {
        display: grid;
        overflow: inherit;
        grid-template-columns: repeat(3, 62px); /* Fixed 3 columns  */
        grid-template-rows: repeat(8, 32px);  /* Fixed 8 rows */
        /* gap: 20px; */
    }

    .skill-left {
        background-image: url("/ui/174-0-dark.png");
        background-repeat: no-repeat;
        background-size: contain;
        display: flex;
        justify-content: center;
        align-items: center;
        width: 31px;
        height: 32px;
        padding: 0;
        margin: 0;
    }

    .skill-bottom {
        width: 62px;
        padding: 0 4px;
        box-sizing: border-box;
        position: absolute;
        bottom: 1px;
    }

    .progress-bar {
        height: 2px;
        width: 100%;
        /* transform: scaleX(.51383266); */
        transform-origin: left;
        /* transform: translateZ(0px); */
    }

    .current-level {
        /* transform: translate(calc(5% - 2px),-60px); */
        transform: translate(0px, 6px);
        text-align: left;
    }

    .base-level {
        /* transform: translate(calc(-5% - 1px),5px); */
        transform: translate(-4px, 6px);
        text-align: right;
    }

    .skill-right {
        background-image: url("/ui/175-0-dark.png");
        background-repeat: no-repeat;
        background-size: contain;
        position: relative;
        width: 31px;
        height: 32px;
        padding: 0;
        margin: 0;
    }

    img {
        /* display: block; */
        /* height: auto; */
        overflow-clip-margin: content-box;
        overflow: clip;
    }

    @font-face {
        font-family: rstiny;
        src: url("/fonts/runescape-tiny.ttf")
    }

    .skill {
        
        background-color: #333;  /* Dark background */
        /* padding: 1px; */
        border-radius: 8px;
        /* border: 1px solid #7d3636; */
        text-align: center;
        display: flex;
        position: relative;
        transform: translateZ(1px);
        /* margin: .5px; */
        /* display: inline; */
    }

    /* .skill-name { */
        /* font-size: 1.2em; */
        /* font-weight: bold; */
    /* } */

    #parent-box {
        font-family: rstiny;
        font-size: 16px;
        color: #ff0;
        /* width: 45%; */
        /* height: 300px; */
        display: block;
    }
</style>
