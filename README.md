# Maubot quick start

To me the [matrix](https://matrix.org/) [maubot](https://github.com/maubot/maubot) was surprisingly hard to get up and running, even with the most trivial setup. This is an attempt to boil down the things I needed to do into a sweet linear step-by-step process to get a simple bot up and running. This guide assumes a system with the following dependencies available:

- Docker
- Sqlite3

1. Create a matrix account specifically for the bot, for instance using the [element app](https://element.io/). From now on let's refer to your bot user id as MXID (e.g. @mybot:matrix.org) and assume you are using the homeserver matrix.org.

2. Create and enter a directory for your bot:

```
mkdir maubot && cd maubot
```

3. Run maubot via docker for the first time. This will create the necessary file structure including the standard configuration file:

```
docker run --name maubot -p 29316:29316 -v $PWD:/data:z dock.mau.dev/maubot/maubot:latest
```

4. Edit the configuration file `config.yaml`.

    1. Add an admin user under "admins". The cleartext password will be encrypted and overwritten when the bot starts. The admins block should look something like this:

    ```
    admins:
        root: ''
        youruser: 'yourpassword'
    ```

    2. Configure the registration_secret block, this will enable us to obtain an access token later:

    ```
    registration_secrets:
        matrix.org:
            # Client-server API URL
            url: https://matrix-client.matrix.org
            # registration_shared_secret from synapse config
            secret: synapse_shared_registration_secret
    ```

5. Restart the container:

```
docker start maubot
```

6. Obtain an access token for the bot:

    1. Log in with the admin user (e.g. youruser) and password, when asked for server use `localhost:29316`:

    `docker exec -it maubot /usr/bin/python3 -m maubot.cli login`

    2. Request an access token, fill in homeserver matrix.org and the username part from "@<username>:matrix.org":

    `docker exec -it maubot /usr/bin/python3 -m maubot.cli auth`

    Take note of the resulting device id DEVICE_ID and access token ACCESS_TOKEN.

7. Stop the docker container since we are about to update its database:

```
docker stop maubot
```

8. Start sqlite3 to assign device id and access token, remember to replace variable names with your specific values. Without modifications `sudo` is needed for sqlite to be able to write to the database:

```sudo sqlite3 maubot.db
   sqlite> UPDATE client SET device_id='<DEVICE_ID>' WHERE id='<MXID>';
   sqlite> UPDATE client SET device_id='<DEVICE_ID>', access_token='<ACCESS_TOKEN>' WHERE id='<MXID>';
   sqlite> .quit
```

9. Download an example maubot plugin, let's use the [dice bot](https://github.com/maubot/dice) in this example. Download the lastest release build (.mbp file) from [https://github.com/maubot/dice/releases] and let's call it `dice.mbp`.

10. Restart maubot:

```
docker start maubot
```

11. Open the maubot admin UI: [http://localhost:29316/\_matrix/maubot/#/](http://localhost:29316/_matrix/maubot/#/).

12. Add a client: Fill in User ID with your bot MXID, select your homeserver from the dropdown and a display name for the bot. Fill in access token then click 'Create'.

13. Add a plugin: upload the `dice.mbp` file.

14. Add an instance: Enter an ID of your choice, select primary user (mybot) and type (the plugin). Click 'Create'.

15. The bot should now be up and running and connected to its homeserver. Try inviting it to your room via its MXID and send a chat message containing "!roll" to the room. If everything works the bot should roll a dice for you and respond with the outcome.

## Troubleshooting

If the bot joins your room when it's invited but doesn't respond to the !roll command, try creating an unencrypted room and invite the bot to it. If the command works in that room, double-check that the sqlite updates were done correctly.

## References

- https://docs.mau.fi/maubot/usage/setup/docker.html
- https://docs.mau.fi/maubot/usage/basic.html
- https://docs.mau.fi/maubot/usage/encryption.html
