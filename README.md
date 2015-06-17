A chat bot for Pokémon Showdown. This bot has a number of commands, some helpful and some less so, as well as the capability to mod. It only reacts to basic offences such as flooding/caps/stretching.

Installation

Pokémon Showdown Bot requires node.js to run. This bot has not been tested on every node.js version possible, but has the same version requirements as Pokémon Showdown: either v0.6.3 through v0.8.22, or v0.10.2 and up. Install node.js if you don't have it yet, try the last stable version.

Next up is cloning this bot. This can be done in two ways: cloning it via git or downloading as ZIP. Downloading it as ZIP is the easy and lazy way, but is a lot less handy to update than cloning this repository.

To install dependencies, run:

npm install
Copy config-example.js to config.js and edit the needed variables. To change the commands that the bot responds to, edit commands.js.

Now, to start the bot, use:

node main.js
Some information will be shown, and will automatically join the room(s) you specified if no error occurs.

If any of your commands rely on information normally sent to the Pokémon Showdown Client (e.g. functions that execute based on PM that aren't directly related to normal commands), you will need to make changes to the message method in parser.js under the appropriate case.
