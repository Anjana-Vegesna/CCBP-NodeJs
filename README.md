# CCBP-NodeJs
CricketTeam Solution


const express = require('express')
const app = express()
const path = require('path')
const {open} = require('sqlite')
const sqlite3 = require('sqlite3')
app.use(express.json())

let db = null

const dbPath = path.join(__dirname, 'cricketTeam.db')

const initializeDbAndServer = async () => {
  try {
    db = await open({
      filename: dbPath,
      driver: sqlite3.Database,
    })

    app.listen(3000, () => {
      console.log('Server Running at http://localhost:3000/')
    })
  } catch (e) {
    console.log(`DB Error: ${e.message}`)
    process.exit(1)
  }
}
initializeDbAndServer()

const convert = dbObject => {
  return {
    playerId: dbObject.player_id,
    playerName: dbObject.player_name,
    jerseyNumber: dbObject.jersey_number,
    role: dbObject.role,
  }
}

// get players list
app.get('/players/', async (request, response) => {
  const getPlayersQuery = `SELECT *
    FROM 
     cricket_team
   ;`
  const playersList = await db.all(getPlayersQuery)
  response.send(playersList.map(eachPlayer => convert(eachPlayer)))
})

// add new player
app.post(' /players/', async (request, response) => {
  const playerDetails = request.body
  const {playerName, jerseyNumber, role} = playerDetails
  const addplayerQuery = `
    INSERT INTO
      cricket_team (player_name, jersey_number, role)
    VALUES
     ("${playerName}",
        ${jerseyNumber},
        "${role}");
     `
  const dbResponse = await db.run(addplayerQuery)
  const playerId = dbResponse.lastId
  response.send('Player Added to Team')
})

//get player
app.get('/players/:playerId/', async (request, response) => {
  const {playerId} = request.params

  const getplayerQuery = `
    SELECT *
    FROM
      cricket_team
    WHERE
     player_id = ${playerId};`

  const player = await db.get(getplayerQuery)

  response.send(convert(player))
})

//update player
app.put('/players/:playerId', async (request, response) => {
  const {playerId} = request.params
  const playerDetails = request.body
  const {playerName, jerseyNumber, role} = playerDetails
  const updateplayerQuery = `
    UPDATE
      cricket_team
    SET
      player_name  = "${playerName}",
        jersey_number = ${jerseyNumber},
       role =  "${role}");
    WHERE
      player_id = ${playerId};
     `
  const dbResponse = await db.run(updateplayerQuery)
  const playerId = dbResponse.lastId
  response.send('Player Added to Team')
})

//delete player
app.delete('/players/:playerId/', async (request, response) => {
  const {playerId} = request.params

  const deleteplayerQuery = `
    DELETE
    FROM
      cricket_team
    WHERE
     player_id = ${playerId};`

  const player = await db.run(deleteplayerQuery)

  response.send('Player Removed')
})

module.exports = app
