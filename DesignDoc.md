---
geometry: margin=1in
---
# Web Checkers Design Documentation

# Team Information
* Team name: kk D
* Team members
    * Shawn Carpenter
    * Jae Yang
    * Alex Mechler
    * Will Kim

## Executive Summary

Web Checkers is a web based implementation of the ever present game Checkers built on the Spark web
framework in Java 8

### Purpose

Our goal for this project is to have a functional system for playing checkers in a web based environment
that allows for unique username players to challenge and play through a game of checkers. The largest
goals of our project are to make sure that a complete game of checkers can be started and played with
another player, making sure that the game adheres to the official American rules of checkers. Another goal of
our project is to give the user an easy experience when using our technology, making each piece of
functionality simple to use.

### Glossary and Acronyms

| Term | Definition |
|------|------------|
| AI | Artificial Intelligence |
| UI | User Interface |


## Requirements

### Definition of MVP

- Users must sign in with a unique valid username.
	* Users cannot use a username that is already signed in.
- Users that have signed in need to be able to sign out.
	* Users must have a way of releasing their usernames so that others may user them.
- Signed in users must be able to play a game of checkers with another signed in user.
	* When a user clicks on another user who is not already in a game, both will be brought into a new game, with the initiator playing as red.
- The game of checkers that is played by two users must adhere to the American rules of checkers.
	* This is enforced by the server. The server governs everything from movement to game over conditions.
- Users must be able to make a move, and then have the option to reconsider before ending the turn.
	* Users must be alerted if their move is invalid, with explanation as to why it is not a valid move.
	* If a move is valid, the user will also have the option to back their turn up, one step at a time. 
	* After a user has made a valid move, they will have the option to submit their turn, committing their move to the board and allowing their opponent to go.
- Users must be able to resign from a game, causing the game to end for both players.
	* Just like in real life, you are allowed to forfeit.

### MVP Features
- Player Sign-In
- Start A Game
- Epic: Piece Movement
  * Regular Move
  * Jump Move
  * King Move
  * King Jump Move
- Crowning
- Player Resigns
- Undo
- Game Over

### Roadmap of Enhancements
1. Permanent registration of usernames
	* Users must be able to register for a username and password combination, so that no one may use someone else's username.
	* Users must sign-in with an already registered username/password combination.
2. Spectator Mode
	* Users must be able to view other player's games as they progress, without impacting the current state of the game.
	* Users who are spectating a game are still available to play, as they are not currently in a game.

## Application Domain

Webcheckers extends solely into the domain of things required to play a game a checkers, that is,
two players, a board, rules, and the pieces required to play the game.
![Domain Analysis](diagrams/domain-analysis.png?raw=true "Figure 1: Domain Analysis")

### Overview of Major Domain Areas

   | Term | Definition |
   |------|------------|
   | Game | The Game is what the Players are playing, inside of a game is the board, which contains all of the Pieces in play. Players manipulate the Pieces through Moves.|
   | Player | The Player is what controls the game, and can challenge others to a game. A singular Player represents a single user on the webcheckers site. |
   | Board | The board is what stores the Pieces. Players make Moves that change the internal state of the board. The board is an 8 x 8 array, with each position in the array being a single position on the board. |
   | Piece |  The piece is what is moved by the moves, can be Red or White, and a Single or King. Single Pieces have different movement rules than King Pieces. King Pieces are allowed to move backwards while single pieces are not. |
   | Move |  A Move is what the players use in order to change the state of the board. Players make one or more moves a turn.|
   | Rules |  The rules are what govern what is legal or illegal for the game. Web Checkers follows the American Checkers rules. Rules include things like Single Pieces may only move forward on the diagonal, and a piece that makes it to the opposite end becomes a King Piece. |

## Architecture 

### Summary
The WebCheckers webapp uses a Java-based web server and the Spark web micro framework and the FreeMarker template engine for handling requests and responses. The Server and client communicate by using standard HTTP GET and POST requests, as well as AJAX route.
![Architecture Diagram](diagrams/arch-diagram.png?raw=true "Figure 2: Architecture Diagram")


### Overview of User Interface

The applications user interface consists of many POST and GET routes that transition from one page to another.

The WebServer class handles requests in Spark using the embedded Jetty web service.

The UI uses the BoardView, Row, Space, and Piece classes to display the board to each user. Unlike the Board class, the BoardView class is rotated to how each player would see the board if they were actually sitting in front of a checkers board. Views are also employed to make sure that players who are just spectating cannot actually influence the game. 

Messages are passed from the server to the UI to pass along statuses for things like move validation and backing up moves

![UI State Chart](diagrams/ui-state.png?raw=true "Figure 3: UI State Chart")

### Tier Application
  The application tier is responsible for user interaction with our WebCheckers Application.
  This includes all and any logic that is relevant application-wide.
  Our application tier contains a PlayerLobby component that handles all users trying to sign in, sign out, all logged-in users, and player count.

  - PlayerLobby Component
    * It must ensure the validity of username of users.
    * If valid, it must be able to register a user and add their username and password to a hashmap.
    * If a user is registered, it must be able to sign in a user and add them to a set of logged in user.
    * In addition, it must be able to sign out players and release their username.

![Registration Sequence Diagram](diagrams/registration-sequence.png?raw=true "Figure 4: Sequence Diagram for a successful registration")

  - GameLobby Component:
    * It is used for storage of all current games.
    * It is able to add, remove and retrieve queried games.

### Tier Model
  The model tier manages our domain entities and any logic pertaining to our domains.
  This includes but is not limited to validation of Model rules, maintaining state, and processing any commands or requests by a user.
  Our model tier contains many components that store the entities of a Checkers game and how users can view them.

  - Board
    * Representation of a checkers board
    * Stores the state of a board and availabilities of spaces.
    * It can return two unique BoardViews depending on the color of each player.
    * Has the ability to return the valid moves that can be made by each player.

  - BoardView
    * After being passed in a Board, it creates a view of iterables which the front-end can parse to create a view of the board.
    * Each board view contains a iterable structure of rows, and each row contains iterable structure of spaces.

  - Color
    * Enumeration to hold the different types of players in Checkers, RED or WHITE.

  - Game
    * Contains the players, current players turn, last validated move and the board.
    * It is responsible passing the validation of a move and storing if it is successful.

  - Message
    * Message to be used with the game template file.

  - Message Type
    * Enumeration of the type of message, either an error or an info.

  - Move
    * Represents a move made by a player.
    * It stores the location of the coordinates of where it started and where it ended.

  - Piece
    * A Piece is responsible for storing what Type(SINGLE or KING), and what color it is.
    * It is responsible for validating move when a given a move.

  - Player
    * Player is responsible for storing each individual as a user.
    * Has the capabilities to be associated with a game.

  - Position
    * Represents a position(Coordinates) on the Board.

  - Row
    * Representation of a row on a checkerboard.
    * Creates Iterable of Spaces given a board.

  - Space
     * A representation of a space on a checkerboard.
     * Responsible for showing availability for piece placement and location for piece movement.

  - Type
    * Enumeration to indicate the type(SINGLE vs KING) of Piece.

  - ViewMode
    * Enumeration to indicate what type(PLAY, SPECTATOR, REPLAY) the player is currently in.

### Tier UI
  The UI queries the Application/Model tier if it is necessary to convey information to the user.
  In addition, it initiates the process of requests and commands received from the user.
  Our UI processes the requests for Sign-in/out, returning to home, and the validity of moves made by the user.

  - GetGameRoute
    * The UI Controller to GET the Game page.
    * Asks the session for the current player and sets two selected users to the same game and generates BoardViews for each player.
    * Once a game has ended, then the players will be redirected to the home page, and told who won.

  - GetHomeRoute
    * The UI Controller to GET the Home page.
    * Renders the home page with a player list if a user is signed in or a number of players if user is not signed in.

  - GetSignRoute
    * The UI Controller to GET the signin page.
    * Renders the sign-in page where a user can sign in for a username

  - GetSignOutRoute
    * The UI Controller to GET the home page after signing out.
    * Redirect and render the homepage after a user signs-out.

  - PostCheckTurn
    * The POST route for CheckTurn, which checks which player's turn it currently is.
    * Checks whether it is your turn or not.

  - PostSigninRoute
    * The UI Controller to POST the signin page.
    * Sign the user into webcheckers if possible, and render the home page if signin is successful,
    * Return to the signin page if there is any problems.

  - PostValidateMove
    * The UI Controller to POST move
    * Check if the move attemped is valid
    
  - PostSubmitTurnRoute
    * The UI controller to POST submitting a turn
    * Submit the valid move that was made, and switch the active player.

  - PostResign
    * The UI controller to POST resigning
    * Removes the player from the game, and then sends the opposing player back to the home screen.
    * Only the current player can resign a game.

  - WebServer
    * The server that initializes the set of HTTP request handlers.

### Model Sub-Systems

## Sub-system Game
This section describes the detail design of sub-system Game.

### Purpose of the sub-system
The purpose of the game subsystem is to represent a single game that is played by two players, on a single board.It    includes the two players, a board representation, and contains this information in the game representation. As the game of checkers is played, the game sub-system is responsible for handling the actions that are performed upon the game by the players, and is responsible for acting as a container for this information.

### Static models

![Class structure diagram of the Game System](diagrams/subsystem-game-uml.png?raw=true "Figure 5: Class Structure of the Game Subsystem")

![State Chart for the Game System](diagrams/game-state.png?raw=true "Figure 6: State Chart of the Game Subsystem")

![Sequence Diagram of Starting a Game](diagrams/game-sequence.png?raw=true "Figure 7: Sequence Diagram of Starting Game")


### UI Sub-Systems

## Sub-system BoardView

This section describes the detail design of sub-system BoardView.

### Purpose of the sub-system
The BoardView sub-system is responsible for giving a representation of a board to the ui in order to be displayed. It    contains the boardview, which contains rows, which have spaces, which can be occupied by spaces. The boardview is able to be iterated over, in order for the ui to properly display the pieces on the board. The boardviews are the top level of this subsystem, and hold all of the information, and are able to generated by the board representation from the game sub-system.


### Static models
![Class Structure model for the Boardview Subsystem](diagrams/subsystem-board-uml.png?raw=true "Figure 8: Class Structure of the BoardView Subsystem")

### Code Metrics

  - Chidamber-Kemerer metrics
    * Average - 9, Player - 29, Game - 26

      PlayerLobby - 20, WebServer - 17, Board - 15

      GameLobby - 15, Message - 12, MessageType - 11

      GetGameRoute - 11, GetSpectateRoute - 10, Piece - 10

    * These are the classes that demonstrate higher than average coupling in our system, some of which such as Web Server make sense as they are used by all routes in the system, but others do not make sense. In the future we should analyze the classes with high coupling that should not have high coupling, such as GetGameRoute, and look into ways to make sure that the system is not overly dependent on these classes. This most likely occurred due to lack of design ahead of time for many of the classes, leading to patchwork solutions being implemented when new ideas were introduced, which increases the coupling of the system.

  - Complexity metrics
    * Board.ValidateMove - 8, PostCheckTurn.handle - 4

      GetSpectateRoute.handle - 4, PostRegisterRoute.handle - 4

    * The classes included were registered as having too high of complexity, but we believe that it is okay that these classes have high complexity. Though we may take a look when reformatting the project, we think that the high complexities for these methods are fine because they have to filter through many different cases, which requires them to have multiple paths. Many of the paths result from edge cases that the methods handle, which leads to higher complexities for those methods.

  - Javadoc coverage metrics
    * First definitely add documentation for methods that have none. Review methods and attributes with low amounts of documentation, and make the decision on whether the documentation needs to be expanded to better encompass what the purpose of the method or attribute is. Tests should include further documentation on the purpose behind why we are testing, and what are we testing.

  - Lines of code metrics
    * Average lines of code is 594.

      UI tier has 1187 lines of code.

      Model tier has 879 lines of code.

      Application tier has 224 lines of code.

      The UI tier has a lot more lines of code that the model or application tier, which could mean that too much responsibility is being assigned to the UI tier instead of the model or application. But, there are a lot of requirements to properly implement the UI tier, which includes communication between the frontend and the backend as well as supporting the display of a game board. For sprint 3, we moved classes from the Model tier to the UI tier because we believed that the responsibilities of those classes better complemented the UI tier.. As for the application tier, it only contains the GameLobby and the PlayerLobby class, which do not have as large of a responsibility as the other tiers.

  - Martin package metrics
    * It makes sense that the UI tier has high efferent coupling because it uses model and application tier components. The afferent coupling is a bit high but the UI tier is used by the frontend, which brings up the afferent coupling value. The model and application tiers have reasonable afferent and efferent couplings. As for instabilities, it is reasonable for the UI tier to have higher instability than the Model and the Application tier because it has higher numbers of afferent and efferent coupling.