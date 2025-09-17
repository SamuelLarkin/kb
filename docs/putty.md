# Putty
## Making a Tunnel

Right-click with the mouse in the window's title bar.

<img width="362" height="526" alt="image" src="https://github.com/user-attachments/assets/db80720a-263b-4950-acdb-53c24066f357" />

Then click `Change Settings... > Connection > SSH > Tunnels`.

Choose a `Source port` like `2024` and a `Destination` `localhost:2044`.

**CLICK** `Add` **THEN** `Apply`

Note that the ports can be different but it is simpler to remember if you use the same port number in both the `Source port` and the `Destination`.

<img width="452" height="438" alt="image-1" src="https://github.com/user-attachments/assets/c5d83445-8006-41c8-a501-50d807f6de40" />

Your tunnel should be operational.  Now use a browser and type `localhost:2024` or change the port to whatever you chose.

Of course, you need to have `tensorboard --port=2024 --bind_all --logdir=. --window_title=FMR` running.

Note that `2024` here is the same as the `Destination` port you chose in `putty`.
