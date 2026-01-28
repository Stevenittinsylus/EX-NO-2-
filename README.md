## EX. NO:2 IMPLEMENTATION OF PLAYFAIR CIPHER

 

## AIM:
 

 

To write a C program to implement the Playfair Substitution technique.

## DESCRIPTION:

The Playfair cipher starts with creating a key table. The key table is a 5×5 grid of letters that will act as the key for encrypting your plaintext. Each of the 25 letters must be unique and one letter of the alphabet is omitted from the table (as there are 25 spots and 26 letters in the alphabet).

To encrypt a message, one would break the message into digrams (groups of 2 letters) such that, for example, "HelloWorld" becomes "HE LL OW OR LD", and map them out on the key table. The two letters of the diagram are considered as the opposite corners of a rectangle in the key table. Note the relative position of the corners of this rectangle. Then apply the following 4 rules, in order, to each pair of letters in the plaintext:
1.	If both letters are the same (or only one letter is left), add an "X" after the first letter
2.	If the letters appear on the same row of your table, replace them with the letters to their immediate right respectively
3.	If the letters appear on the same column of your table, replace them with the letters immediately below respectively
4.	If the letters are not on the same row or column, replace them with the letters on the same row respectively but at the other pair of corners of the rectangle defined by the original pair.
## EXAMPLE:
![image](https://github.com/Hemamanigandan/EX-NO-2-/assets/149653568/e6858d4f-b122-42ba-acdb-db18ec2e9675)

 

## ALGORITHM:

## STEP-1: Read the plain text from the user.

## STEP-2: Read the keyword from the user.

## STEP-3: Arrange the keyword without duplicates in a 5*5 matrix in the row order and fill the remaining cells with missed out letters in alphabetical order. Note that ‘i’ and ‘j’ takes the same cell.

## STEP-4: Group the plain text in pairs and match the corresponding corner letters by forming a rectangular grid.

## STEP-5: Display the obtained cipher text.




Program:
```
#include <stdio.h>
#include <string.h>
#include <ctype.h>

#define SIZE 5

char matrix[SIZE][SIZE];

// Remove duplicate characters from keyword and fill matrix
void generateMatrix(const char key[]) {
    int alphabet[26] = {0};
    int row = 0, col = 0;

    // Process key first (ignore non-alpha, map J -> I)
    for (int i = 0; key[i] && row < SIZE; i++) {
        if (!isalpha((unsigned char)key[i])) continue;
        char ch = toupper((unsigned char)key[i]);
        if (ch == 'J') ch = 'I';
        if (!alphabet[ch - 'A']) {
            matrix[row][col++] = ch;
            alphabet[ch - 'A'] = 1;
            if (col == SIZE) { col = 0; row++; }
        }
    }

    // Fill remaining letters A-Z (skip J)
    for (char ch = 'A'; ch <= 'Z' && row < SIZE; ch++) {
        if (ch == 'J') continue;
        if (!alphabet[ch - 'A']) {
            matrix[row][col++] = ch;
            alphabet[ch - 'A'] = 1;
            if (col == SIZE) { col = 0; row++; }
        }
    }
}

// Locate position of a character in matrix
void findPosition(char ch, int *row, int *col) {
    if (ch == 'J') ch = 'I';
    for (int i = 0; i < SIZE; i++) {
        for (int j = 0; j < SIZE; j++) {
            if (matrix[i][j] == ch) {
                *row = i;
                *col = j;
                return;
            }
        }
    }
    // Should never happen for letters A-Z excluding J
    *row = -1; *col = -1;
}

// Encrypt digraphs (two-letter array, not null-terminated)
void encryptDigraph(char a, char b, char out[3]) {
    int r1, c1, r2, c2;
    findPosition(a, &r1, &c1);
    findPosition(b, &r2, &c2);

    if (r1 == -1 || r2 == -1) { // safety check
        out[0] = out[1] = 'X';
        out[2] = '\0';
        return;
    }

    if (r1 == r2) {
        out[0] = matrix[r1][(c1 + 1) % SIZE];
        out[1] = matrix[r2][(c2 + 1) % SIZE];
    } else if (c1 == c2) {
        out[0] = matrix[(r1 + 1) % SIZE][c1];
        out[1] = matrix[(r2 + 1) % SIZE][c2];
    } else {
        out[0] = matrix[r1][c2];
        out[1] = matrix[r2][c1];
    }
    out[2] = '\0';
}

// Prepare plaintext into digraphs following Playfair rules.
// Returns number of digraphs produced and fills digraphs[][2].
int prepareText(const char *input, char digraphs[][2], int maxpairs) {
    // 1) Build cleaned string: only letters, uppercase, J->I
    char cleaned[512];
    int clen = 0;
    for (int i = 0; input[i] && clen < (int)sizeof(cleaned) - 1; i++) {
        if (!isalpha((unsigned char)input[i])) continue;
        char ch = toupper((unsigned char)input[i]);
        if (ch == 'J') ch = 'I';
        cleaned[clen++] = ch;
    }
    cleaned[clen] = '\0';

    // 2) Form digraphs
    int len = 0;
    int i = 0;
    while (i < clen && len < maxpairs) {
        char a = cleaned[i];
        if (i + 1 >= clen) {
            // last single letter -> pad with 'X'
            digraphs[len][0] = a;
            digraphs[len][1] = 'X';
            len++;
            i++;
        } else {
            char b = cleaned[i + 1];
            if (a == b) {
                // same letters in pair -> pair a with 'X' and advance by 1
                digraphs[len][0] = a;
                digraphs[len][1] = 'X';
                len++;
                i++; // only move one position
            } else {
                // normal pair
                digraphs[len][0] = a;
                digraphs[len][1] = b;
                len++;
                i += 2;
            }
        }
    }

    return len;
}

// Display matrix (for debugging)
void displayMatrix() {
    printf("Playfair Matrix:\n");
    for (int i = 0; i < SIZE; i++) {
        for (int j = 0; j < SIZE; j++) {
            printf("%c ", matrix[i][j]);
        }
        printf("\n");
    }
}

int main() {
    const char key[] = "KEYWORD";       // change as needed
    const char plaintext[] = "shankarswetha"; // change as needed

    generateMatrix(key);
    displayMatrix();

    // Prepare digraphs
    char digraphs[256][2];
    int count = prepareText(plaintext, digraphs, 256);

    printf("\nPlaintext Digraphs:\n");
    for (int i = 0; i < count; i++) {
        printf("%c%c ", digraphs[i][0], digraphs[i][1]);
    }
    printf("\n\nEncrypted Text: ");

    // Encrypt and print
    char encryptedPair[3];
    char encryptedText[1024] = ""; // ensure enough space
    for (int i = 0; i < count; i++) {
        encryptDigraph(digraphs[i][0], digraphs[i][1], encryptedPair);
        strcat(encryptedText, encryptedPair);
        printf("%s ", encryptedPair);
    }

    printf("\n\nFinal Encrypted Output: %s\n", encryptedText);
    return 0;
}




```





Output:


<img width="542" height="462" alt="image" src="https://github.com/user-attachments/assets/63ebe6cb-b226-43a2-b85e-7af580580782" />
