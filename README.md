# SimpleVoteSystem

This plugin is a voting plugin designed to be put on a BungeeCord server, its purpose is to provide compatibility for voting sites, which do not support votifier.


## What voting sites are currently supported?

- https://serveur-prive.net/
- https://www.serveurs-minecraft.org/
- https://www.serveursminecraft.org/
- https://www.liste-serveurs-minecraft.org/

## As a server owner, how can I install the plugin on my server?

### I advise you to divide the installation process in two steps:

#### The first one is to configure the database
So you have to go to the config.yml file generated by the plugin at the first start, and look at the `database` key:
```yaml
database:
  host: localhost
  port: 3306
  database: db
  username: user 
  password: passw
```

You will have to change the accesses indicated here by the real accesses to your database.

#### The second step is to configure the votes websites
You have to look at the `websites` key of the configuration file, the typical configuration for a voting site looks like this:
```yaml
serveur_PriveNet:
  enabled: true
  votePage: 'https://serveur-prive.net/minecraft/ivernya-3422'
  serverId: '5GHP0copFW7AlCi'
```
There are 3 particular values:
- `enabled`: this allows you to enable or disable one of the voting sites supported by the plugin.
- `votePage`: This is absolutely visual, it is the voting link that your players will find when they drag the mouse over the name of the voting site in the voting command.
- `serverId`: This value is very important, it's the ID of your server, or the vote token depending on the voting site, which will allow the plugin to detect the vote with the API of the voting website.

##### Then you are good to go.

#### The votes cleaner
Now maybe you are wondering what is the `useCleaner` key in configuration, it allows your server, if set to true, to clean all the vote counters every first day of a new month.

## As a vote website owner, how can I add my website?
You need to add a class in the websites package that extends from the AbstractHasVoted abstract class, it contains the following abstract methods which you will have to implement:

```java 
public abstract int hasVoted(ProxiedPlayer player); //The player has voted? If yes, then returns the time in seconds before the next vote, if not, returns -1

public abstract String getWebsiteName(); //The name of the site as it should be in the configuration, and in the database

public abstract String getUserFriendlyName(); //The name as it will appear in the configuration's placeholders, and in the voting command

public abstract String getUrl(String serverId, String playerIp); //The Voting API link, which will be formatted with the player IP and the server ID or token on your voting site.
```

The important thing to remember that you have to implement is the hasVoted abstract method, it's really simple, it should typically look like this if your API is ideal for the system:

```java
    @Override
    public int hasVoted(ProxiedPlayer player) {
        JSONObject result = ReadersUtil.readJsonFromUrl(getUrl(getServerIdForWebsite(), player.getAddress().getHostString()));
        if (result.getInt("status") == 1) {
            return result.getInt("nextvote");
        } else {
            return -1;
        }
    }
```


If the IP in question has already voted on your website, you have to return the time in seconds left before the next vote.

If you don't detect any vote on that IP, then return -1.
