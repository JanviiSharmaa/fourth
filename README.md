#include <iostream>
#include <string>
#include <cctype>
#include <stdexcept>

using namespace std;

// Token types
enum TokenType { NUMBER, PLUS, MINUS, MUL, DIV, LPAREN, RPAREN, END };

// Token structure
struct Token {
    TokenType type;
    double value; // Only used for NUMBER
};

// Lexer: Converts string into tokens
class Lexer {
    string text;
    size_t pos = 0;
    char currentChar;

public:
    Lexer(const string& input) : text(input) {
        currentChar = text[pos];
    }

    void advance() {
        pos++;
        if (pos < text.length())
            currentChar = text[pos];
        else
            currentChar = '\0';
    }

    void skipWhitespace() {
        while (isspace(currentChar)) advance();
    }

    double getNumber() {
        string result;
        while (isdigit(currentChar) || currentChar == '.') {
            result += currentChar;
            advance();
        }
        return stod(result);
    }

    Token getNextToken() {
        while (currentChar != '\0') {
            if (isspace(currentChar)) {
                skipWhitespace();
                continue;
            }

            if (isdigit(currentChar)) {
                return Token{NUMBER, getNumber()};
            }

            if (currentChar == '+') {
                advance();
                return Token{PLUS, 0};
            }
            if (currentChar == '-') {
                advance();
                return Token{MINUS, 0};
            }
            if (currentChar == '*') {
                advance();
                return Token{MUL, 0};
            }
            if (currentChar == '/') {
                advance();
                return Token{DIV, 0};
            }
            if (currentChar == '(') {
                advance();
                return Token{LPAREN, 0};
            }
            if (currentChar == ')') {
                advance();
                return Token{RPAREN, 0};
            }

            throw runtime_error("Invalid character");
        }

        return Token{END, 0};
    }
};

// Parser/Evaluator
class Interpreter {
    Lexer lexer;
    Token currentToken;

public:
    Interpreter(Lexer lex) : lexer(lex) {
        currentToken = lexer.getNextToken();
    }

    void eat(TokenType type) {
        if (currentToken.type == type) {
            currentToken = lexer.getNextToken();
        } else {
            throw runtime_error("Syntax error");
        }
    }

    double factor() {
        if (currentToken.type == NUMBER) {
            double val = currentToken.value;
            eat(NUMBER);
            return val;
        } else if (currentToken.type == LPAREN) {
            eat(LPAREN);
            double result = expr();
            eat(RPAREN);
            return result;
        } else if (currentToken.type == MINUS) {
            eat(MINUS);
            return -factor();
        }
        throw runtime_error("Unexpected token in factor");
    }

    double term() {
        double result = factor();
        while (currentToken.type == MUL || currentToken.type == DIV) {
            if (currentToken.type == MUL) {
                eat(MUL);
                result *= factor();
            } else {
                eat(DIV);
                double divisor = factor();
                if (divisor == 0) throw runtime_error("Division by zero");
                result /= divisor;
            }
        }
        return result;
    }

    double expr() {
        double result = term();
        while (currentToken.type == PLUS || currentToken.type == MINUS) {
            if (currentToken.type == PLUS) {
                eat(PLUS);
                result += term();
            } else {
                eat(MINUS);
                result -= term();
            }
        }
        return result;
    }

    double evaluate() {
        return expr();
    }
};

int main() {
    cout << "Simple Arithmetic Expression Evaluator (type 'exit' to quit)\n";
    string input;

    while (true) {
        cout << ">>> ";
        getline(cin, input);
        if (input == "exit") break;

        try {
            Lexer lexer(input);
            Interpreter interpreter(lexer);
            double result = interpreter.evaluate();
            cout << "Result: " << result << endl;
        } catch (const exception& e) {
            cerr << "Error: " << e.what() << endl;
        }
    }

    return 0;
}
