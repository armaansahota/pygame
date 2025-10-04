import os
import subprocess
import sys

# --- Configuration ---
# Set the directory where your games will be stored.
# IMPORTANT: Create this directory in the same location as this script.
GAMES_DIRECTORY = "games"
ALLOWED_EXTENSIONS = ['.py']
PYTHON_EXECUTABLE = sys.executable # Uses the same Python that runs this script

def setup_directory():
    """Ensures the 'games' directory exists."""
    if not os.path.exists(GAMES_DIRECTORY):
        os.makedirs(GAMES_DIRECTORY)
        print(f"Created games directory: '{GAMES_DIRECTORY}'")
        print("Please place your game scripts (e.g., 'my_game.py') inside this folder.")

def get_available_games():
    """Scans the games directory for runnable scripts."""
    game_files = []
    # os.listdir gets all files and directories
    for filename in os.listdir(GAMES_DIRECTORY):
        # os.path.splitext separates the file name from its extension
        name, ext = os.path.splitext(filename)
        # Check if the extension is allowed and it's a file, not a folder
        if ext in ALLOWED_EXTENSIONS and os.path.isfile(os.path.join(GAMES_DIRECTORY, filename)):
            # Only store the base name (e.g., 'my_game' instead of 'my_game.py')
            game_files.append(name)
    return sorted(game_files)

def display_menu(games):
    """Prints the console menu to the screen."""
    print("\n" + "="*40)
    print("👾 RASPBERRY PI CONSOLE LAUNCHER 🕹️")
    print("="*40)
    
    if not games:
        print("No games found! Please upload a '.py' game script to the 'games' folder.")
    else:
        print("Available Games:")
        # Enumerate games to assign a number to each
        for i, game_name in enumerate(games):
            print(f"  [{i+1}] {game_name}")
    
    print("-" * 40)
    print("  [U] How to Upload a Game")
    print("  [Q] Quit Console")
    print("-" * 40)

def upload_instructions():
    """Provides simple instructions on 'uploading' a game."""
    print("\n" + "="*40)
    print("      HOW TO UPLOAD A GAME")
    print("="*40)
    print("1. **Find the script**: Locate your Pygame project file (e.g., 'my_awesome_game.py').")
    print(f"2. **Copy the script**: Paste this file into the '{GAMES_DIRECTORY}' folder.")
    print("3. **Restart the launcher**: Quit (Q) and run this console script again.")
    print("   Your game will now appear in the list.")
    print("\n* Ensure your game file can be run by typing: `python my_awesome_game.py`")
    input("\nPress Enter to return to the main menu...")

def run_game(game_file_path):
    """Executes the selected game script."""
    print(f"\n🚀 Launching {os.path.basename(game_file_path)}...")
    
    # We use subprocess.run to execute the external Python script
    # The command is: 'python /path/to/game_script.py'
    try:
        # Popen is used for more control, but run is simpler for a basic launch
        result = subprocess.run(
            [PYTHON_EXECUTABLE, game_file_path],
            check=True, # Raise an error if the process fails
            # stdout/stderr=subprocess.PIPE can capture output if needed
        )
        print(f"\n🏁 Game finished with return code {result.returncode}.")
    except subprocess.CalledProcessError as e:
        print(f"\n❌ ERROR: The game crashed! Check your game's code.")
        print(f"Details: {e}")
    except FileNotFoundError:
         print(f"\n❌ ERROR: Python executable not found. Check your environment.")
    
    input("\nPress Enter to return to the console menu...")

def main():
    """The main loop of the console launcher."""
    setup_directory()
    
    while True:
        available_games = get_available_games()
        display_menu(available_games)
        
        choice = input("Enter choice (number, U, or Q): ").strip().upper()
        
        # Quit option
        if choice == 'Q':
            print("\nGoodbye! Console shutting down.")
            break
        
        # Upload instructions
        elif choice == 'U':
            upload_instructions()
        
        # Game selection
        elif choice.isdigit():
            try:
                game_index = int(choice) - 1
                if 0 <= game_index < len(available_games):
                    selected_game_name = available_games[game_index]
                    # Construct the full path to the game file (e.g., 'games/my_game.py')
                    game_file_path = os.path.join(GAMES_DIRECTORY, selected_game_name + ALLOWED_EXTENSIONS[0])
                    run_game(game_file_path)
                else:
                    print("Invalid game number. Please try again.")
            except ValueError:
                print("Invalid input. Please enter a number, 'U', or 'Q'.")
        else:
            print("Invalid command. Please try again.")

if __name__ == "__main__":
    # Ensure all print statements happen immediately, which is useful for terminal programs
    sys.stdout.reconfigure(line_buffering=True)
    main()
