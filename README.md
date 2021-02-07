#include <iostream>
#include <string>
using namespace std;

const int BOARD_WIDTH = 15;//15-wide
const int BOARD_HEIGHT = 10;//10-height
const int SHIP_TYPES = 5;//5-ships

const char isWATER = 24; //ASCII Code
const char isHIT = 'H';
const char isSHIP = 'S';
const char isMISS = '0';
bool gameRunning = false;

struct POINT {
	//A location on the grid defined
	//by X(horizontal) Y(vertical) coordinates
	int X;
	int Y;
};

struct SHIP {
	//Ship name
	string name;
	//Total points on the grid
	int length;
	//Coordinates of those points
	POINT onGrid[5]; //0-4 max length of biggest ship
	//Whether or not those points are a "hit"
	bool hitFlag[5];
}ship[SHIP_TYPES];

struct PLAYER {
	char grid[BOARD_WIDTH][BOARD_HEIGHT];
}player[3]; //Using player's 1 & 2

enum Direction {HORIZONTAL,VERTICAL};
struct PLACESHIPS {
	Direction direction;
	SHIP shipType;
};

//Functions
void LoadShips();
void ResetBoard();
void DrawBoard(int);
PLACESHIPS UserInputShipPlacement();
bool UserInputAttack(int&,int&,int);
bool GameOverCheck(int);

int main()
{
	LoadShips();
	ResetBoard();

	//"PLACE SHIPS" phase of gam
	//Loop through each player
	for (int currPlayer=1; currPlayer<3; ++currPlayer)
	{
		//Loop through each ship type to place
		for (int thisShip=0; thisShip<SHIP_TYPES; ++thisShip)
		{
			//Display gameboard for player
			system("cls");
			DrawBoard(currPlayer);
			//Game instructions
			cout << "\n";
			cout << "INSTRUCTIONS (Player " << currPlayer << ")"<<endl;
			cout << "You are placing your ship"<<endl;
			cout << "0-Horizontal,1-Vertical, X (top-row) Y (left-side) "<<endl;
			cout << "For example 1 5 5 this would place a ship beginning at X:5 and Y:5 going vertical"<<endl;
			cout << "Ship to place: " << ship[thisShip].name << " which has a length of " << ship[thisShip].length  << endl;
			cout << "Where do you want it placed? ";
			PLACESHIPS aShip;
			aShip.shipType.onGrid[0].X = -1;
			while (aShip.shipType.onGrid[0].X == -1)
			{
				aShip = UserInputShipPlacement();
			}

			//Combine user data with "this ship" data
			aShip.shipType.length = ship[thisShip].length;
			aShip.shipType.name = ship[thisShip].name;

			//Add the FIRST grid point to the current player's game board
			player[currPlayer].grid[aShip.shipType.onGrid[0].X][aShip.shipType.onGrid[0].Y] = isSHIP;

			//Determine ALL grid points based on length and direction
			for (int i=1; i<aShip.shipType.length; ++i)
			{
				if (aShip.direction == HORIZONTAL){
					aShip.shipType.onGrid[i].X = aShip.shipType.onGrid[i-1].X+1;
					aShip.shipType.onGrid[i].Y = aShip.shipType.onGrid[i-1].Y; }
				if (aShip.direction == VERTICAL){
					aShip.shipType.onGrid[i].Y = aShip.shipType.onGrid[i-1].Y+1;
					aShip.shipType.onGrid[i].X = aShip.shipType.onGrid[i-1].X; }

				//Add the REMAINING grid points to our current players game board
				player[currPlayer].grid[aShip.shipType.onGrid[i].X][aShip.shipType.onGrid[i].Y] = isSHIP;
			}
		}
	}

	//FINISHED WITH "PLACE SHIPS" PHASE
	gameRunning = true;
	int thisPlayer = 1;
	do {
		//Because we are ATTACKING now, the
		//opposite players board is the display board
		int enemyPlayer;
		if (thisPlayer == 1) enemyPlayer = 2;
		if (thisPlayer == 2) enemyPlayer = 1;
		system("cls");
		DrawBoard(enemyPlayer);

		//Get attack coords from this player
		bool goodInput = false;
		int x,y;
		while (goodInput == false) {
			goodInput = UserInputAttack(x,y,thisPlayer);
		}

		//Check board; if a ship is there = HIT if not = MISS
		if (player[enemyPlayer].grid[x][y] == isSHIP) player[enemyPlayer].grid[x][y] = isHIT;
		if (player[enemyPlayer].grid[x][y] == isWATER) player[enemyPlayer].grid[x][y] = isMISS;

		//Check to see if the game is over
		//If 0 is returned, nobody has won yet
		int aWin = GameOverCheck(enemyPlayer);
		if (aWin != 0) {
			gameRunning = false;
			break;
		}
		//Alternate between each player as we loop back around
		thisPlayer = (thisPlayer == 1) ? 2 : 1;
	} while (gameRunning);

	system("cls");
	cout << "CONGRATULATIONS!!!  PLAYER " << thisPlayer << " YOU WON!";

	system("pause");
	return 0;
}


bool GameOverCheck(int enemyPLAYER)
{
	bool winner = true;
	//Loop through enemy board
	for (int w=0; w<BOARD_WIDTH; ++w){
			for (int h=0; h<BOARD_HEIGHT; ++h){
				//If any ships remain game is not over
				if (player[enemyPLAYER].grid[w][h] = isSHIP)
					{
						winner = false;
						return winner;
					}
		}
    }
	//If we get here somebody won
	return winner;
}


bool UserInputAttack(int& x, int& y, int theplayer)
{
	cout  << "PLAYER " << theplayer << " ENTER COORDINATES TO ATTACK: ";
	bool goodInput = false;
	cin >> x >> y;
	if (x<0 || x>=BOARD_WIDTH) return goodInput;
	if (y<0 || y>=BOARD_HEIGHT) return goodInput;
	goodInput = true;
	return goodInput;
}

PLACESHIPS UserInputShipPlacement()
{
	int d, x, y;
	PLACESHIPS tmp;
	//if it is wrong input
	tmp.shipType.onGrid[0].X = -1;
	cin >> d >> x >> y;
	if (d!=0 && d!=1) return tmp;
	if (x<0 || x>=BOARD_WIDTH) return tmp;
	if (y<0 || y>=BOARD_HEIGHT) return tmp;
	//Good input
	tmp.direction = (Direction)d;
	tmp.shipType.onGrid[0].X = x;
	tmp.shipType.onGrid[0].Y = y;
	return tmp;
}

void LoadShips()
{
	//Sets the default data for the ships
	ship[0].name = "|Little|"; ship[0].length = 2;
	ship[1].name = "|Normal|"; ship[1].length = 3;
	ship[2].name = "|Normal|"; ship[2].length = 3;
	ship[3].name = "|Average|"; ship[3].length = 4;
	ship[4].name = "|Large|"; ship[4].length = 5;
}
void ResetBoard()
{
	//Loop through each player
	for (int plyr=1; plyr<3; ++plyr)
	{
		//For each grid point, set contents to 'water'
		for (int w=0; w<BOARD_WIDTH; ++w){
			for (int h=0; h<BOARD_HEIGHT; ++h){
				player[plyr].grid[w][h] = isWATER;
		}
		}
	}
}

void DrawBoard(int thisPlayer)
{
	//Draws the board for a player (thisPlayer)
	cout << "PLAYER " << thisPlayer << "'s GAME BOARD"<<endl;
	cout << "----------------------"<<endl;

	//Loop through top row and number columns
	cout << "   ";
	for (int w=0; w<BOARD_WIDTH; ++w) {
		if (w < 10)
			//Numbers only 1 character long, add two spaces after
			cout << w << "  ";
		else if (w >= 10)
			//Numbers 2 characters long, add only 1 space after
			cout << w << " ";
	}
	cout << endl;

	//Loop through each grid point and display to console
	for (int h=0; h<BOARD_HEIGHT; ++h){
		for (int w=0; w<BOARD_WIDTH; ++w){

			//If this is the FIRST (left) grid point, number the grid first
			if (w==0)
            cout << h << " ";
			//If h was 1 character long, add an extra space to keep numbers lined up
			if (w<10 && w==0)
			cout << " ";
			//Display contents of this grid (if game isnot running yet, we are placing ships
			if (gameRunning == false)
            cout << player[thisPlayer].grid[w][h] << "  ";
			//Don't show ships, BUT show damage if it's hit
			if (gameRunning == true && player[thisPlayer].grid[w][h] != isSHIP)
			{
                cout << player[thisPlayer].grid[w][h] << "  ";
            }
			else if (gameRunning == true && player[thisPlayer].grid[w][h] == isSHIP)
			{
			    cout << isWATER << "  ";
            }
			//If we have reached the border line
			if (w == BOARD_WIDTH-1)
                cout << endl;
		}
	}
}
