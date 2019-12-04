/*
 * assignment1.cpp
 * Class: CS 2337
 * By: Gabe Vogel
 * UID: gjv180000
 */
#include <iostream>
#include <string>
#include <fstream>
#include <cstring>
#include <algorithm>
#include <cctype>
using namespace std;
enum direction {LEFT_RIGHT, RIGHT_LEFT, DOWN, UP, RIGHT_DOWN, RIGHT_UP, LEFT_DOWN, LEFT_UP};
const int MAX = 50;

struct wordGame
{
	int version;
	int numberRows;
	int numberColumns;
	char puzzle[MAX][MAX];

};

struct wordFind
{
	string word;
	bool found;
	int row;
	int column;
	direction where;
	int foundCount;
};

// find the word that is part of the wordFind structure inside the wordGame structure.
// If the word is found the wordFind structure will be updated.
void findWord(wordGame &game, wordFind &theFind);
// read the puzzle from the input file and update the wordGame structure.
bool readPuzzle(wordGame &game, string inputFileName);
// display the contents of the puzzle
void displayPuzzle(wordGame &game);

int main()
{
	//Defining variables + structures
	string fileName;
	string puzzleName;
	wordGame game;
	wordFind find;
	ifstream filer;

	//Getting the name of the puzzle from user input and checking if it exists.
	cin >> puzzleName;
	if(readPuzzle(game, puzzleName))
	{
		cout << "The puzzle from file \"" << puzzleName << "\"" << endl;
	}
	else
	{
		cerr << "The puzzle file \"" << puzzleName << "\" could not be opened or is invalid" << endl;
		abort();
	}
	displayPuzzle(game);
	cin >> fileName;
	//Getting the name of the word list from user input and checking if it exists.
	filer.open(fileName);
	if(!filer)
	{
		cerr << "The puzzle file \"" << fileName << "\" could not be opened or is invalid" << endl;
		abort();
	}

	//Checks if each word in the file matches a location in the puzzle, if so it outputs the location and direction
	while(filer >> fileName)
	{
		for(int i = 0; i < static_cast<int>(fileName.length()); i++)
		{
			fileName[i] = toupper(fileName[i]);
		}
		find.word = (fileName);
		findWord(game, find);
		if(find.foundCount == 0)
		{
			cout << "The word " << find.word << " was not found" << endl;
		}
		//Goes through each possible case in the direction enum, and outputs the applicable direction to console.
		else if(find.foundCount == 1)
		{
			cout << "The word " << find.word << " was found at (" << find.row + 1 << ", " << find.column + 1 << ")";
			switch(find.where){
			case LEFT_RIGHT :
				cout << " - right" << endl;
				break;
			case RIGHT_LEFT:
				cout << " - left" << endl;
				break;
			case DOWN:
				cout << " - down" << endl;
				break;
			case UP:
				cout << " - up" << endl;
				break;
			case RIGHT_DOWN:
				cout << " - right/down" << endl;
				break;
			case RIGHT_UP:
				cout << " - right/up" << endl;
				break;
			case LEFT_DOWN:
				cout << " - left/down" << endl;
				break;
			case LEFT_UP:
				cout << " - left/up" << endl;
				break;
			}
		}
		else
		{
			cout << "The word " << find.word << " was found " << find.foundCount << " times" << endl;
		}
		find.foundCount = 0;


	}
	filer.close();
	return 0;
}

bool readPuzzle(wordGame &game, string inputFileName)
{
	ifstream file;
	string text;
	//int i is defined to track which line of input is currently being read. This allows the program to know when to assign rows and columns, and is also used in a for loop to parse through the puzzle array.
	int i = 0;
	int nums = 0;
	game.version = 2;
	file.open(inputFileName);
	if(!file)
	{
		return false;
	}
	while (file >> text)
	{
	   // Checks if the number of rows is between 1 and 50. If so, assigns it to game.numberRows
	   if(i == 0)
	   {
		   if(std::stoi(text) < 1 || std::stoi(text) > 50){
			   return false;
		   }
		   game.numberRows = stoi(text);

	   }
	   // Checks if the number of columns is between 1 and 50. If so, assigns it to game.numberColumns
	   else if(i == 1)
	   {
		   if(std::stoi(text) < 1 || std::stoi(text) > 50)
		   {
			   return false;
		   }
		   game.numberColumns = stoi(text);
	   }
	   // Creates the game.puzzle matrix.
	   else
	   {
		   for(int k = 0; k < game.numberColumns; k++)
		   {
			   game.puzzle[i - 2][k] = toupper(text[k]);
			   nums += 1;
			   }
		   }
	   i += 1;
	}

	//Checks if there are enough values in the input file for all of the rows and columns specified by the file
	if(nums!= (game.numberRows * game.numberColumns))
	{
		return false;
	}
	file.close();
	return true;
}

void displayPuzzle(wordGame &game)
{
	//Goes through each element in the puzzle array and prints it to console
	for(int i = 0; i < game.numberRows; i++)
	{
		for(int k = 0; k < game.numberColumns; k++)
		{
			cout << game.puzzle[i][k];
		}
		cout << endl;
	}
}
void findWord(wordGame &game, wordFind &theFind)
{
	theFind.foundCount = 0;
	for(int i = 0; i < game.numberRows; i++)
	{
		for(int k = 0; k < game.numberColumns; k++)
		{
			if(game.puzzle[i][k] == theFind.word[0])
			{
				// Used to keep track of how many letters in a direction match with the appropriate letter in the game word
				int loopTracker = 0;
				//Temporary variables used to track which element of the puzzle array to access, without changing the actual values of i and k.
				int tempi = i;
				int tempk = k;
				bool a = true;
				if(game.puzzle[i - 1][k] == theFind.word[1] &&  (i  >= static_cast<int>(theFind.word.length()) - 1))
				{
					while(a && loopTracker < static_cast<int>(theFind.word.length() - 1))
					{
						if(game.puzzle[tempi - 1][tempk] == theFind.word[loopTracker + 1])
						{
							tempi--;
							loopTracker++;
						}
						else
						{
							a = false;
						}

					}
					if((loopTracker + 1) == static_cast<int>(theFind.word.length()))
					{
						theFind.where = UP;
						theFind.foundCount += 1;
						theFind.row = i;
						theFind.column = k;
						theFind.found = true;
					}
					//Resets loopTracker to its original state as to not interfere with the other direction cases.
					loopTracker =0;
				}
				//Resets the temporary variables
				tempi = i;
				tempk = k;
				a = true;
				if((game.puzzle[i - 1][k - 1] == theFind.word[1]) && (k >= static_cast<int>(theFind.word.length() -1)) && (static_cast<int>(theFind.word.length()) -1 <= i ))
				{
					while(a && loopTracker < static_cast<int>(theFind.word.length() - 1))
					{
						if(game.puzzle[tempi - 1][tempk - 1] == theFind.word[loopTracker + 1])
						{
							tempk--;
							tempi--;
							loopTracker++;
						}
						else
						{
							a = false;
						}
					}
					if((loopTracker + 1) == static_cast<int>(theFind.word.length()))
					{
						theFind.where= LEFT_UP;
						theFind.foundCount += 1;
						theFind.row = i;
						theFind.column = k;
						theFind.found = true;

					}
					loopTracker =0;
				}
				a = true;
				tempi = i;
				tempk = k;
				if(game.puzzle[i - 1][k + 1] == theFind.word[1] && (static_cast<int>(theFind.word.length()) -1 <= i ) && ((game.numberColumns - k -1) >= static_cast<int>(theFind.word.length() -1)))
				{
					while(a && loopTracker < static_cast<int>(theFind.word.length()) - 1)
					{
						if(game.puzzle[tempi - 1][tempk + 1] == theFind.word[loopTracker + 1])
						{
							tempk++;
							tempi--;
							loopTracker++;
						}
						else
						{
							a = false;
						}
					}
					if((loopTracker + 1) == static_cast<int>(theFind.word.length()))
					{
						theFind.where = RIGHT_UP;
						theFind.foundCount += 1;
						theFind.row = i;
						theFind.column = k;
						theFind.found = true;
					}
					loopTracker = 0;

				}
				tempi = i;
				tempk = k;
				a = true;
				if(game.puzzle[i][k + 1] == theFind.word[1] && game.numberColumns - k >= static_cast<int>(theFind.word.length()))
				{
					while(a && loopTracker < static_cast<int>(theFind.word.length() - 1))
					{
						if(game.puzzle[tempi][tempk + 1] == theFind.word[loopTracker + 1])
						{
							tempk++;
							loopTracker++;
						}
						else
						{
							a = false;
						}
					}
					if((loopTracker + 1) == static_cast<int>(theFind.word.length()))
					{
						theFind.where = LEFT_RIGHT;
						theFind.foundCount += 1;
						theFind.row = i;
						theFind.column = k;
						theFind.found = true;
					}
					loopTracker = 0;
				}
				a = true;
				tempi = i;
				tempk = k;
				if(game.puzzle[i][k - 1] == theFind.word[1] && (k >= static_cast<int>(theFind.word.length() -1)))
				{
					while(a && (loopTracker < static_cast<int>(theFind.word.length() - 1)))
					{
						if(game.puzzle[tempi][tempk - 1] == theFind.word[loopTracker + 1])
						{
							tempk--;
							loopTracker++;
						}
						else
						{
							a = false;
						}

					}
					if(((loopTracker + 1) == static_cast<int>(theFind.word.length())))
					{
						theFind.foundCount += 1;
						theFind.where= RIGHT_LEFT;
						theFind.row = i;
						theFind.column = k;
						theFind.found = true;

					}
					loopTracker =0;
				}
				a = true;
				tempi = i;
				tempk = k;
				if((game.puzzle[i + 1][k + 1] == theFind.word[1] && game.numberColumns - k >= static_cast<int>(theFind.word.length()) && game.numberRows >= static_cast<int>(theFind.word.length())))
				{
					while(a && loopTracker < static_cast<int>(theFind.word.length() - 1))
					{
						if(game.puzzle[tempi + 1][tempk + 1] == theFind.word[loopTracker + 1])
						{
							tempk++;
							tempi++;
							loopTracker++;
						}
						else
						{
							a = false;
						}
					}
					if((loopTracker + 1) == static_cast<int>(theFind.word.length()))
					{
						theFind.foundCount += 1;
						theFind.where= RIGHT_DOWN;
						theFind.row = i;
						theFind.column = k;
						theFind.found = true;

					}
					loopTracker =0;
				}
				a = true;
				tempi = i;
				tempk = k;
				if(game.puzzle[i + 1][k] == theFind.word[1] && game.numberRows >= static_cast<int>(theFind.word.length()))
				{
					while(a && loopTracker < static_cast<int>(theFind.word.length() - 1))
					{
						if(game.puzzle[tempi + 1][tempk] == theFind.word[loopTracker + 1])
						{
							tempi++;
							loopTracker++;
						}
						else
						{
							a = false;
						}
					}
					if((loopTracker + 1) == static_cast<int>(theFind.word.length()))
					{
						theFind.where = DOWN;
						theFind.foundCount += 1;
						theFind.row = i;
						theFind.column = k;
						theFind.found = true;
					}
					loopTracker = 0;
				}
				a = true;
				tempi = i;
				tempk = k;
				if(game.puzzle[i + 1][k - 1] == theFind.word[1] && game.numberRows >= static_cast<int>(theFind.word.length() && (loopTracker < static_cast<int>(theFind.word.length() - 1))))
				{
					while(a && loopTracker < static_cast<int>(theFind.word.length()) - 1)
					{
						if(game.puzzle[tempi + 1][tempk - 1] == theFind.word[loopTracker + 1])
						{
							tempk--;
							tempi++;
							loopTracker++;
						}
						else
						{
							a = false;
						}
					}
					if((loopTracker + 1) == static_cast<int>(theFind.word.length()))
					{
						theFind.where = LEFT_DOWN;
						theFind.foundCount += 1;
						theFind.row = i;
						theFind.column = k;
						theFind.found = true;
					}
					loopTracker = 0;

				}
				tempi = i;
				tempk = k;
				a = true;

			}
		}
	}
}
