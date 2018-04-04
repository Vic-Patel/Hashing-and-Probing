#include <iostream>
#include <fstream>
#include <string>
#include <cstdlib>
#include <cstring>
#include <vector>
using namespace std;


unsigned long hash(string str) {
	unsigned long hash = 5381;
	int c, count = 0;
	while (str[count]) {
		c = str[count++];
		hash = ((hash << 5) + hash) + c;
	}
	return hash;
}
int lp_probing(string *arr, string s, int size, int &uniqueWords,  int &collisions) {

	int index = 0;
	for (int x = 0; x < size; x++) {
		index = (hash(s) + x) % size;
		if (arr[index] == "" || arr[index] == s) {
			if (arr[index] != s) {
				uniqueWords++;
			}
			
			return index;
		}
		collisions++;
	}
}
int qp_probing(string *arr, string s, int size, int &uniqueWords, int &collisions) {
	int index = 0;
	int u = 0;
	for (int x = 0; x < size; x++) {
		u = x;
		index = (hash(s) + u*u) % size;
		if (arr[index] == "" || arr[index] == s) {
			if (arr[index] != s) {	
				uniqueWords++;
			}
			return index;
		}
		collisions++;
	}
}

//   H(x) = H1(x) + i H2(x)
int dh_probing(string *arr, string s, int size, int &uniqueWords, int &collisions, int Int) {
	int index = 0;
	int index2 = 0;
	int index3 = 0;
	for (int x = 0; x < size; x++) {
		if (x == 0) {
			index = hash(s) % size;
			index3 = index;
		}
		else if (x > 0) {
			index3 = index + x * index2;
			index3 %= size;
		}
		if (arr[index3] == "" || arr[index3] == s) {
			if (arr[index3] != s) {
				uniqueWords++;
			}
			return index3;
		}
		index2 = Int - (index % Int);
		collisions++;
	}
}

typedef struct node {
	int line;
	string strin;
	node *next;
	node*prev;
} node;

class line {
public:
	void PrintLine(node*ptr, string word) {
		int x = 0;
		cout << word << " appears on lines";
		while (ptr != NULL) {
			if (x == 0) {
				if (word == ptr->strin) {
					cout << "[" << ptr->line;
					x++;
				}
			}
			else {
				if (word == ptr->strin) {
					cout << "," << ptr->line;
				}
			}
			ptr = ptr->next;
		}
		if (x > 0) {
			cout << "]" << endl;
		}
		else {
			cout << " appears on lines " << "[]" << endl;
		}
		return;
	}

	node * lineList(int countLine, int countWord, node *ptr, string strin) {
		node *newNode = new node;
		newNode->strin = strin;
		newNode->line = countLine;
		newNode->next = NULL;
		if (countWord == 0) {
			head = ptr;
		}
		tail = newNode;
		if (ptr == NULL) {
			return newNode;
		}
		ptr->next = tail;
		tail->prev = ptr;
		return tail;
	}
private:
	node* head;
	node* tail;
};

int main(int argc, char **argv) {
	node *getLine = NULL;
	node *getLine2 = NULL;
	line Line;
	ifstream inFS;
	inFS.open(argv[1]);
	if (!inFS.is_open()) {
		cout << "Error: File did not open." << endl;
		return -1;
	}

	char letters;
	string strin;
	int collisions = 0;
	int countLines = 1;
	int countWords = 0;
	int size = atoi(argv[3]);
	int index = 0;
	int uniqueWords = 0; // put into lphashing pls
	string words[size];
	for (int x; x < size; x++) {
		words[x] = "\0";
	}
	string word = argv[4];
	
	int dhInt;
	while (inFS.get(letters)) {
		if (letters == '\n') {
			countLines++; //  <-------------- This is the problem
		}
		 if (!isspace(letters)) {
			strin.push_back((letters));
		}
	
		 else  if (strin != "") {
			if (countWords == 0) {
				getLine = Line.lineList(countLines, countWords, getLine, strin);
				getLine2 = getLine;
			}
			else {
				getLine2 = Line.lineList(countLines, countWords, getLine2, strin);
			}

			if (word == "lp") {
				index = lp_probing(words, strin, size, uniqueWords, collisions);
			}
			else if (word == "qp") {
				index = qp_probing(words, strin, size, uniqueWords, collisions);
			}
			else if (word == "dh") {
				dhInt = atoi(argv[5]);
				index = dh_probing(words, strin, size, uniqueWords, collisions, dhInt);
			}
			words[index] = strin;
			strin.clear();
			countWords++;
			index = 0;
		}
	}
	
		inFS.close();
			cout << "The number of words found in the file was " << countWords << endl;
			cout << "The number of unique words found in the file was " << uniqueWords << endl;
			cout << "The number of collisions was " << collisions << endl << endl;
			collisions = 0;
			ifstream infs;
			infs.open(argv[2]);
			if (!infs.is_open()) {
				cout << "Error: File did not open." << endl;
				return -1;
			}
			string strtemp;
			node* hashy = getLine;
			while (infs.get(letters)) {
				if (!isspace(letters)) {
					strtemp.push_back((letters));
				}
				else if (strtemp != "") {
					Line.PrintLine(hashy, strtemp);
					if (word == "lp") {
					index = lp_probing(words, strtemp, size, uniqueWords, collisions);
					cout << "The search had " << collisions <<" collisions" << endl;
					}
					else if (word == "qp") {
					index = qp_probing(words, strtemp, size, uniqueWords, collisions);
					cout << "The search had " << collisions <<" collisions" << endl;
					}
					else {
					dhInt = atoi(argv[5]);
					index = dh_probing(words, strtemp, size, uniqueWords, collisions, dhInt);
					cout << "The search had " << collisions <<" collisions" << endl;
					}
					cout << endl << endl;
					strtemp.clear();
					collisions = 0;
				}
			}
			
		return 0;
	}
