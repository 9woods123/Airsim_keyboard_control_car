# Welcome to Modified AirSim

This project is based on [AirSim](https://microsoft.github.io/AirSim/), a popular drone simulation platform. The project is developed as a UE4 plugin, creating a GameMode to load the simulation game. However, Microsoft did not provide a simulation with both a car and an aircraft simultaneously. To better adapt to the simulation of target tracking algorithms, we conducted secondary development to create a multiplayer game that allows for simultaneous control of both aircraft and car models.
![image](https://github.com/user-attachments/assets/cca87443-943e-4c83-b719-5c4b63c07b8f)

# Control the car by keyboard W A S D. And control your multicopter by original interface provided by Microsoft.

# Develop UE game in Windows and Package game to Linux

### Windows

#### Build AirSim
Install Visual Studio 2022. Make sure to select Desktop Development with C++ and Windows 10 SDK 10.0.19041 (should be selected by default) and select the latest .NET Framework SDK under the 'Individual Components' tab while installing VS 2022.
Start Developer Command Prompt for VS 2022.
Clone the repo: git clone https://github.com/9woods123/simulator.git, and go the AirSim directory by cd AirSim.

Note: It's generally not a good idea to install AirSim in C drive. This can cause scripts to fail, and requires running VS in Admin mode. Instead clone in a different drive such as D or E.

Run build.cmd from the command line. This will create ready to use plugin bits in the Unreal\Plugins folder that can be dropped into any Unreal project.

### Build Project to Linux Platform

To package the simulation software for Linux, you need to convert the `.uproject` into a C++ project and then recompile it on Linux with the AirSim Plugin, including the dynamic link libraries `libxxx.a`, to achieve cross-compilation. The main logic is to maintain the UE4 Windows project while copying the `libAirLib.a`, `libMavLinkCom.a`, and `librpc.a` files to the appropriate locations since they are required during the UE4 packaging process.

1. Install WSL with the following command:
   ```
   wsl --install -d Ubuntu-18.04
   ```
   Then, restart your machine.

2. After rebooting, execute the command again to create a new user account:
   ```
   wsl --install -d Ubuntu-18.04
   ```

3. Once inside the Ubuntu subsystem, run:
   ```bash
   sudo apt-get update
   sudo apt-get install dos2unix
   cd Airsim/ && dos2unix build.sh && dos2unix setup.sh
   cd Unreal/Environments/Blocks/ && dos2unix clean.sh
   ```
   This converts all necessary files.

4. In the **Windows Developer Command Prompt for VS 2022**, navigate to the AirSim directory and build the project:
   ```cmd
   cd Airsim/ && build.cmd
   ```
   This will generate a `Block.uproject` that can be opened on Windows. Open the `.uproject` file to let UE4 recompile `Block.uproject`.

5. In WSL ubuntu, return to the AirSim directory and set up the Linux environment:
   ```bash
   ./setup.sh  # Install dependencies for compiling AirSim on Linux
   ./build.sh  # Compile AirSim; this will create an AirSim uproject for Ubuntu
   ```

6. However, since you need to handle the `Block.uproject` on Windows, Linux compilation alone isn’t sufficient. The purpose of compiling AirSim on Linux is to obtain a few essential dynamic link libraries (executing `build.sh` will copy some libraries directly to the `AirSim/Unreal/Plugins/AirSim/Source/AirLib/lib` directory). At this point, the `block.uproject` is still configured for Linux.

7. Therefore, re-run:
   ```cmd
   cd Airsim/ && build.cmd
   ```
   Open `Block.uproject` to let UE4 recompile it again on Windows. Then, copy `libMavLinkCom.a` to the `Block/Plugins/AirSim/Source/AirLib/deps/MavLinkCom/lib` directory.

8. If you want to use this plugin, copy the `Plugins` folder to the directory at the same level as the `Content` folder within your project.**After opening the `.uproject`, create a new C++ file (this will trigger UE4 to recompile the Plugins during the packaging process)** . Since the necessary dynamic link libraries have been added to the Plugins folder, packaging should proceed without issues.

# Using 

The drone will be designated as Player 0, and the vehicle will be designated as Player 1 (based on the numbering in UE games). You can use all the features of AirSim, and the vehicle can be controlled via the keyboard. The vehicle’s coordinates can be obtained through RPC communication. Additionally, we have written a ROS node to relay the vehicle’s odometry.
   ```
#include "easytrack/easyTrackRpcClient.hpp"
    std::string ip_address = "127.0.0.1"; 
    uint16_t port = 50502;                 
    float timeout_sec = 5.0f;             

    msr::airlib::easyTrackRpcClient client_easy(ip_address, port,timeout_sec);

    try {
        int32_t player_index = 1;
        msr::airlib::Vector3r position = client_easy.getPlayerPosition(player_index);

        std::cout << "Player Position: " 
                  << "X: " << position.x() 
                  << " Y: " << position.y() 
                  << " Z: " << position.z() << std::endl;
    }
    catch (const std::exception& e) {
        std::cerr << "Error: " << e.what() << std::endl;
    }
   ```

# If you find our work useful we'd love to hear from you. If you use this repositorty as part of your research can you please cite the repository in your work:

@misc{Airsim_keyboard_control_car,
  author = {Yu, Tianyou},
  title = {Airsim\_keyboard\_control\_car},
  year = {2024},
  publisher = {GitHub},
  journal = {GitHub repository},
  howpublished = {\url{https://github.com/9woods123/Airsim_keyboard_control_car}}
}

