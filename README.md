# UMD-Mens-Soccer-Performance-Analysis
Overview

Welcome to Varsity League Soccer StatPad, a comprehensive system designed for athletes, coaches, and spectators of Maryland Men's Soccer. This readme file provides an overview of the project's purpose, objectives, data sources, references, and how to test your project with screenshots.


Purpose

The purpose of this project is to conduct a thorough analysis of the performance of the Maryland Men’s Soccer team in their recent season. It aims to derive valuable insights regarding team dynamics in various match settings and assess individual player performance to understand their impact on overall team success.

Mission Objectives

1. What is the correlation between goals, assists and fouls with player position?

2. How can we analyze the relationship between different match statistics, such as shots on target, fouls, and goals scored, for both home and away teams

3. What is the win/loss statistics for a team playing Home and Away and how much does attendance affect team performance?

4. Is there a correlation between the spectator count and player performance? If yes, which forward player performs the best when there are more spectators watching.



Data Sources

The data for this project was collected from the University of Maryland's '2022 Men’s Soccer Cumulative Statistics' available at [UMD Men's Soccer Stats 2022]
(https://umterps.com/sports/mens-soccer/stats/2022). 

		Example of Data For 1 Match Between Maryland and New Hampshire



Testing Instructions
To test the project, follow these step-by-step instructions:

Step 1: Importing and Running DDL Scripts

1. Drop: Download and Execute the `Project_0504_11_DROP.sql` script to drop any existing database objects related to the project. After executing the DROP code, the results should be displayed in the “Results” Tab, as shown in the attached screenshot below
 
   			FIG1.1						FIG 1.2

As we can see in FIG 1.2 No new Tables are present in the Database				

2. Create: Download and Run the `Project_0504_11_CREATE.sql` script to create the necessary tables and schema for the project. After executing the CREATE code, the results should be displayed in the “Results” Tab, as shown in the attached screenshot below
 
			FIG 2.1				FIG 2.2

As we can see from FIG 2.2 New tables are created in the database






3. Insert: Download and Execute the `Project_0504_11_INSERT.sql` script to populate the tables with data collected from the UMD Men's Soccer Cumulative Statistics. After executing the INSERT code, the results should be displayed in the “Results” Tab, as shown in the attached screenshot below
 
			FIG 3.1					FIG 3.2

As we can see from FIG 3.2 data has been successfully inserted into their respective tables. FIG 3.2 is obtained by using a SELECT statement after inserting the data

Step 2: Importing and Executing DML Queries



4. Objective 1: Import and execute `Project_0504_11_Objective1.sql` script (or) copy paste the code below in a new SQL file to determine the correlation between goals, assists, and fouls with player position.

CODE: 


USE BUDT702_Project_0504_11
DROP VIEW IF EXISTS PlayerStats;

-- Create a view named PlayerStats 
CREATE VIEW PlayerStats AS
SELECT 
    p.playerPosition,
    AVG(p.playerGoalsScored) OVER (PARTITION BY p.playerPosition) AS avgGoals,
    AVG(p.playerAssists) OVER (PARTITION BY p.playerPosition) AS avgAssists,
    AVG(p.playerYellowCardCount + p.playerRedCard) OVER (PARTITION BY p.playerPosition) AS avgFouls
FROM 
    [StatPad.Player] p;

-- Select and aggregate the data from the PlayerStats view based on player position
GO
SELECT ps.playerPosition AS 'Player Position', 
	SUM(ps.avgGoals) AS 'Average Goals', 
	SUM(ps.avgAssists) AS 'Average Assists', 
	SUM(ps.avgFouls) AS 'Average Fouls' FROM PlayerStats ps
WHERE ps.playerPosition IN ('GK', 'DEF', 'MID', 'FWD')
GROUP BY ps.playerPosition

Results and Status Bar: 






5. Objective 2: Import and execute `Project_0504_11_Objective2.sql` script (or) copy paste the code in a new SQL file to analyze the relationship between different match statistics (e.g., shots on target, fouls, goals scored) for both home and away teams.

CODE: 


USE BUDT702_Project_0504_11
SELECT
COALESCE(m.matchId, 'All Matches') AS 'Match Id',
COALESCE(p.teamIdFirst, 'All Teams') AS 'Away Team',
COALESCE(p.teamIdSecond, 'All Teams') AS 'Home Team',
COALESCE(SUM(ms.matchHomeTeamShotsOnTarget), 0) AS 'Home Shots on Target',
COALESCE(SUM(ms.matchAwayTeamShotsOnTarget), 0) AS 'Away Shots on Target',
COALESCE(SUM(ms.matchHomeTeamFouls), 0) AS 'Home Fouls',
COALESCE(SUM(ms.matchAwayTeamFouls), 0) AS 'Away Fouls',
COALESCE(SUM(ms.matchGoalsScored), 0) AS 'Total Goals Scored'
FROM
[StatPad.Match] m
LEFT JOIN
[StatPad.Participate] p ON m.matchId = p.matchId
LEFT JOIN
[StatPad.MatchStats] ms ON m.matchId = ms.matchId
GROUP BY CUBE (m.matchId, p.teamIdFirst, p.teamIdSecond)
ORDER BY [Match Id], [Home Team], [Away Team];








Results and Status Bar: 






6. Objective 3: Win/Loss Statistics: Import and execute `Project_0504_11_Objective3.sql` script (or) copy paste the code in a new SQL file to investigate win/loss statistics for the team playing at home and away, and assess the impact of attendance on team performance.

CODE:
USE BUDT702_Project_0504_11

SELECT 
    t.teamId AS 'Team Id',
    t.teamName AS 'Opposition Team Name',
    SUM(CASE 
            WHEN p.teamIdFirst = t.teamId AND p.homeTeamScore > p.awayTeamScore THEN 1
            WHEN p.teamIdSecond = t.teamId AND p.awayTeamScore > p.homeTeamScore THEN 1
            ELSE 0 
        END) AS 'Maryland Win',
    SUM(CASE 
            WHEN p.teamIdFirst = t.teamId AND p.homeTeamScore < p.awayTeamScore THEN 1
            WHEN p.teamIdSecond = t.teamId AND p.awayTeamScore < p.homeTeamScore THEN 1
            ELSE 0 
        END) AS 'Maryland Loss',
    COUNT(CASE 
            WHEN p.teamIdFirst = t.teamId THEN 1
            WHEN p.teamIdSecond = t.teamId THEN 1
            ELSE NULL 
        END) AS 'Total Matches',
    AVG(m.matchAttendance) AS 'Average Attendance',
    ROUND((SUM(CASE 
                WHEN p.teamIdFirst = t.teamId AND p.homeTeamScore > p.awayTeamScore THEN 
                WHEN p.teamIdSecond = t.teamId AND p.awayTeamScore > p.homeTeamScore THEN 
                ELSE 0 
            END) / CAST(COUNT(CASE 
                                WHEN p.teamIdFirst = t.teamId THEN 1
                                WHEN p.teamIdSecond = t.teamId THEN 1
                                ELSE NULL 
                            END) AS FLOAT)) * 100, 2) AS 'Win Percentage',
    CASE 
        WHEN t.teamId = 'T0001' THEN 1
        ELSE 0 
    END AS 'Maryland Home Game Flag'
FROM 
    [StatPad.Team] t
LEFT JOIN 
    [StatPad.Participate] p ON t.teamId = p.teamIdFirst OR t.teamId = p.teamIdSecond
LEFT JOIN 
    [StatPad.Match] m ON m.matchId = p.matchId
WHERE 
    t.teamId != 'T0001'
GROUP BY 
    t.teamId, t.teamName
ORDER BY 
    'Win Percentage' DESC;





Results and Status Bar:













7. Objective 4: Spectator Influence: Import and execute `Project_0504_11_Objective4.sql` script (or) copy paste the code in a new SQL file to explore the correlation between spectator count and forward player performance, identifying the best-performing forward player in high spectator environments.

CODE:
USE BUDT702_Project_0504_11

-- Calculating FWD players performance based on their G/A and match attendance
SELECT
    p.playerId AS 'Player Id',
    p.playerFirstName AS 'Player First Name',
    p.playerLastName AS 'Player Last Name',
	p.playerPosition AS 'Player Position',
    m.matchId AS 'Match Id',
    m.matchAttendance AS 'Match Attendance',
    pl.minutesPlayed AS 'Minutes Played',
	CAST(
		CASE
			WHEN PL.minutesPlayed > 0 THEN
			-- Formula Used = (((goal * 2 * (match Attendance/1000)) + (assist * 1.5 * (match Attendance/1000))) / minutes played * 90)
				(((pl.playerGoalCount * 2.0 * (m.matchAttendance / 1000)) + (pl.playerAssistCount * 1.5 * (m.matchAttendance / 1000))) / pl.minutesPlayed * 90) 
			ELSE
				0.0  -- default value when minutesPlayed is 0
		END AS DECIMAL(5,2)) AS 'Player Score Correlating With Audience Count' 
FROM
    [StatPad.Match] m
JOIN
    [StatPad.Play] pl ON m.matchId = pl.matchId
JOIN
    [StatPad.Player] p ON pl.playerId = p.playerId
WHERE
    p.playerPosition = 'FWD'  -- Specify the position as FWD
ORDER BY
    'Player Score Correlating With Audience Count'  DESC;


Results and Status Bar:



References: None

