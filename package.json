require("dotenv").config();

const express = require("express");
const {
  Client,
  GatewayIntentBits,
  Collection,
  Events,
  REST,
  Routes,
  ActivityType
} = require("discord.js");

const fs = require("fs");
const path = require("path");

const app = express();
const PORT = process.env.PORT || 10000;

// Render health check
app.get("/", (req, res) => {
  res.send("🐉 The Elite is Online!");
});

app.listen(PORT, () => {
  console.log(`Web server running on port ${PORT}`);
});

const client = new Client({
  intents: [
    GatewayIntentBits.Guilds,
    GatewayIntentBits.GuildMessages,
    GatewayIntentBits.MessageContent,
    GatewayIntentBits.GuildMembers
  ]
});

client.commands = new Collection();

// Load commands
const commands = [];
const commandsPath = path.join(__dirname, "commands");

if (fs.existsSync(commandsPath)) {
  const folders = fs.readdirSync(commandsPath);

  for (const folder of folders) {
    const folderPath = path.join(commandsPath, folder);
    const commandFiles = fs
      .readdirSync(folderPath)
      .filter(file => file.endsWith(".js"));

    for (const file of commandFiles) {
      const filePath = path.join(folderPath, file);
      const command = require(filePath);

      if ("data" in command && "execute" in command) {
        client.commands.set(command.data.name, command);
        commands.push(command.data.toJSON());
      }
    }
  }
}

client.once(Events.ClientReady, async () => {
  console.log(`✅ Logged in as ${client.user.tag}`);

  client.user.setPresence({
    activities: [
      {
        name: "🐉 Dragon Soul",
        type: ActivityType.Playing
      }
    ],
    status: "online"
  });

  try {
    const rest = new REST({ version: "10" }).setToken(process.env.TOKEN);

    console.log("Registering slash commands...");

    await rest.put(
      Routes.applicationCommands(process.env.CLIENT_ID),
      {
        body: commands
      }
    );

    console.log(`✅ Registered ${commands.length} commands.`);
  } catch (err) {
    console.error(err);
  }
});

client.on(Events.InteractionCreate, async interaction => {
  if (!interaction.isChatInputCommand()) return;

  const command = client.commands.get(interaction.commandName);

  if (!command) return;

  try {
    await command.execute(interaction);
  } catch (error) {
    console.error(error);

    if (interaction.replied || interaction.deferred) {
      await interaction.followUp({
        content: "There was an error executing this command.",
        ephemeral: true
      });
    } else {
      await interaction.reply({
        content: "There was an error executing this command.",
        ephemeral: true
      });
    }
  }
});

client.login(process.env.TOKEN);