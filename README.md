# Compilador Kotlin - Implementación de Tipos String y Float

## Introducción

Este proyecto presenta la implementación de un compilador para un subconjunto del lenguaje Kotlin, con especial enfoque en la incorporación de tipos de datos **String** y **Float**. Si bien el compilador maneja estructuras de control tradicionales como bucles `for`, `while` y declaraciones condicionales, el núcleo de esta implementación se centra en la gestión avanzada de cadenas de texto y números en coma flotante.

## Gramática del Lenguaje

El compilador sigue la siguiente gramática EBNF:

```ebnf
Program = TopLevelDeclaration*

TopLevelDeclaration = VarDeclaration
                    | FunDeclaration

Statement = VarDeclaration
          | Assignment
          | PrintStatement
          | IfStatement
          | WhileStatement
          | DoWhileStatement
          | ForStatement
          | ReturnStatement
          | BreakStatement
          | ContinueStatement
          | Block
          | RunBlock
          | ExpressionStatement

VarDeclaration = ("var" | "val") IDENTIFIER ":" Type ["=" Expression] [";"]
FunDeclaration = "fun" IDENTIFIER "(" [Parameters] ")" [":" Type] Block
Parameters = Parameter ("," Parameter)*
Parameter = IDENTIFIER ":" Type

Assignment = IDENTIFIER AssignOperator Expression [";"]
           | IDENTIFIER ("++" | "--") [";"]
           | ("++" | "--") IDENTIFIER [";"]
AssignOperator = "=" | "+=" | "-=" | "*=" | "/=" | "%="

PrintStatement = ("print" | "println") "(" Expression ")" [";"]
IfStatement = "if" "(" Expression ")" Statement ["else" Statement]
WhileStatement = "while" "(" Expression ")" Statement
DoWhileStatement = "do" Statement "while" "(" Expression ")" [";"]
ForStatement = "for" "(" IDENTIFIER "in" Range ")" Statement
ReturnStatement = "return" [Expression] [";"]
BreakStatement = "break" [";"]
ContinueStatement = "continue" [";"]
Block = "{" Statement* "}"
RunBlock = "run" "{" Statement* "}"
ExpressionStatement = (FunctionCallExpression | IDENTIFIER) [";"]

Range = Expression RangeOperator Expression ["step" Expression]
RangeOperator = ".." | "until" | "downTo"

Expression = OrExpression
OrExpression = AndExpression ("||" AndExpression)*
AndExpression = EqualityExpression ("&&" EqualityExpression)*
EqualityExpression = RelationalExpression (("==" | "!=") RelationalExpression)*
RelationalExpression = AdditiveExpression (("<" | "<=" | ">" | ">=") AdditiveExpression)*
AdditiveExpression = MultiplicativeExpression (("+" | "-") MultiplicativeExpression)*
MultiplicativeExpression = UnaryExpression (("*" | "/" | "%") UnaryExpression)*
UnaryExpression = ("!" | "-" | "+" | "++" | "--") UnaryExpression
                | PostfixExpression
PostfixExpression = PrimaryExpression ("++" | "--")*
PrimaryExpression = IDENTIFIER
                  | NUMBER
                  | DECIMAL
                  | STRING
                  | BOOLEAN
                  | ParenthesizedExpression
                  | FunctionCallExpression
                  | RunExpression

ParenthesizedExpression = "(" Expression ")"
FunctionCallExpression = IDENTIFIER "(" [Arguments] ")"
RunExpression = "run" "{" Statement* "}"
Arguments = Expression ("," Expression)*

Type = "Int" | "Float" | "String" | "Boolean" | "Unit"

BOOLEAN = "true" | "false"
NUMBER = DIGIT+
DECIMAL = DIGIT+ "." DIGIT* "f" | DIGIT+ "f"
STRING = '"' CHARACTER* '"'
IDENTIFIER = LETTER (LETTER | DIGIT | "_")*

COMMENT = "//" CHARACTER* NEWLINE | "/*" CHARACTER* "*/"

DIGIT = "0" | "1" | "2" | "3" | "4" | "5" | "6" | "7" | "8" | "9"
LETTER = "a" | "b" | "c" | ... | "z" | "A" | "B" | "C" | ... | "Z" | "_"
CHARACTER = (* cualquier carácter válido *)
NEWLINE = "\n" | "\r\n"
```

```

```
## Implementación de Tipos String y Float

### 1. Clases Elaboradas

La incorporación de tipos String y Float al compilador requirió el desarrollo de clases específicas y la extensión de las estructuras existentes para manejar estos nuevos tipos de datos de manera eficiente y coherente con el diseño del sistema.

#### Tipo String

El manejo de cadenas de texto se implementó a través de varias clases y estructuras especializadas que permiten el procesamiento completo de literales de cadena, desde su reconocimiento léxico hasta su evaluación en tiempo de ejecución.

La clase **StringExp** representa los literales de cadena en el árbol sintáctico abstracto (AST). Esta clase hereda de la clase base `Exp` y encapsula el valor de la cadena como una propiedad de tipo `string`. Su implementación es la siguiente:

```cpp
class StringExp : public Exp
{
public:
    string value;
    StringExp(const string &v);
    int accept(Visitor *visitor);
    ~StringExp();
};
```

El constructor de `StringExp` recibe una referencia constante a un string, lo que garantiza eficiencia en la copia y evita modificaciones accidentales del valor durante la construcción. La función `accept` implementa el patrón Visitor, permitiendo que diferentes tipos de visitadores procesen esta expresión de manera polimórfica.

En el nivel léxico, se implementó el token **STRING_TYPE** para identificar el tipo String en las declaraciones de variables. Este token se reconoce mediante la palabra clave "String" y se utiliza durante el análisis sintáctico para validar las declaraciones de tipo. El scanner también maneja los literales de cadena mediante un análisis especializado que procesa caracteres entre comillas dobles:

```cpp
Token* Scanner::scanString() {
    string text = "";
    consumeChar(); // Consume la comilla inicial
    
    while (current != '"' && current != '\0') {
        if (current == '\\') {
            consumeChar();
            switch (current) {
                case 'n': text += '\n'; break;
                case 't': text += '\t'; break;
                case 'r': text += '\r'; break;
                case '\\': text += '\\'; break;
                case '"': text += '"'; break;
                default: text += current; break;
            }
        } else {
            text += current;
        }
        consumeChar();
    }
    
    if (current == '"') {
        consumeChar(); // Consume la comilla final
        return new Token(Token::STRING, text, line, col);
    }
    return new Token(Token::ERR, text, line, col);
}
```

Esta función maneja secuencias de escape como `\n`, `\t`, `\r`, `\\` y `\"`, proporcionando funcionalidad completa para caracteres especiales dentro de las cadenas. El manejo de errores incluye la detección de cadenas no terminadas, retornando un token de error cuando no se encuentra la comilla de cierre.

#### Tipo Float

La implementación de números en coma flotante se centra en la clase **DecimalExp**, que maneja tanto la representación interna como las operaciones aritméticas de precisión flotante.

```cpp
class DecimalExp : public Exp
{
public:
    double value;
    DecimalExp(double v);
    int accept(Visitor *visitor);
    ~DecimalExp();
};
```

La clase utiliza el tipo `double` de C++ para almacenar valores de precisión doble, proporcionando mayor rango y precisión que los tipos float estándar. El constructor acepta un valor double y lo almacena directamente, manteniendo la precisión original del literal analizado.

El token **DECIMAL** se encarga del reconocimiento léxico de literales flotantes. El scanner implementa un análisis sofisticado que reconoce múltiples formatos de números flotantes:

```cpp
Token* Scanner::scanNumber() {
    string num = "";
    bool isFloat = false;
    
    // Escanear dígitos antes del punto decimal
    while (isdigit(current)) {
        num += current;
        consumeChar();
    }
    
    // Verificar punto decimal
    if (current == '.') {
        isFloat = true;
        num += current;
        consumeChar();
        
        // Escanear dígitos después del punto decimal
        while (isdigit(current)) {
            num += current;
            consumeChar();
        }
    }
    
    // Verificar sufijo 'f' para float
    if (current == 'f' || current == 'F') {
        isFloat = true;
        consumeChar();
    }
    
    if (isFloat) {
        double value = stod(num);
        return new Token(Token::DECIMAL, to_string(value), line, col);
    } else {
        int value = stoi(num);
        return new Token(Token::NUMBER, to_string(value), line, col);
    }
}
```

Esta implementación reconoce formatos como `3.14`, `2.5f`, `42f`, y `0.001`, convirtiendo automáticamente el texto a valores numéricos apropiados. La función `stod` (string to double) maneja la conversión con manejo automático de errores de formato.

### 2. Operaciones Implementadas

La implementación de operaciones para String y Float requirió la extensión significativa del sistema de evaluación de expresiones binarias y la incorporación de lógica de conversión de tipos automática.

#### Operaciones con String

Las operaciones con cadenas se implementaron principalmente a través de la sobrecarga semántica del operador de suma (`+`) para realizar concatenación. Esta funcionalidad se maneja en la clase `BinaryExp` cuando se detectan operandos de tipo String:

```cpp
int EvalVisitor::visit(BinaryExp* exp) {
    int leftType = exp->left->accept(this);
    string leftStr = (leftType == 5) ? lastString : to_string(lastInt);
    
    int rightType = exp->right->accept(this);
    string rightStr = (rightType == 5) ? lastString : to_string(lastInt);
    
    if (exp->op == PLUS_OP && (leftType == 5 || rightType == 5)) {
        // Concatenación de strings
        lastString = leftStr + rightStr;
        lastType = 5; // String type
        return 5;
    }
    
    if ((exp->op == EQ_OP || exp->op == NE_OP) && 
        (leftType == 5 || rightType == 5)) {
        // Comparación de strings
        bool result = (exp->op == EQ_OP) ? 
                     (leftStr == rightStr) : 
                     (leftStr != rightStr);
        lastInt = result ? 1 : 0;
        lastType = 3; // Boolean type
        return 3;
    }
    
    // ... resto de operaciones
}
```

Esta implementación permite concatenación polimórfica, donde strings pueden concatenarse con otros strings o con representaciones de cadena de otros tipos. Las comparaciones de igualdad y desigualdad se implementan usando los operadores nativos de string de C++, proporcionando comparación lexicográfica completa.

La integración con `print()` y `println()` se maneja en la clase `PrintStatement`:

```cpp
void EvalVisitor::visit(PrintStatement* stm) {
    int type = stm->e->accept(this);
    
    switch (type) {
        case 5: // String
            cout << lastString;
            break;
        case 1: // Int
            cout << lastInt;
            break;
        case 2: // Float
            cout << lastFloat;
            break;
        case 3: // Boolean
            cout << (lastInt ? "true" : "false");
            break;
    }
    
    if (stm->isPrintln) {
        cout << endl;
    }
}
```

#### Operaciones con Float

Las operaciones aritméticas con números flotantes se implementaron con soporte completo para conversiones automáticas de tipo y manejo de precisión. El sistema detecta automáticamente cuando una operación involucra al menos un operando flotante y promueve toda la operación a aritmética de punto flotante:

```cpp
int EvalVisitor::visit(BinaryExp* exp) {
    int leftType = exp->left->accept(this);
    double leftVal = (leftType == 2) ? lastFloat : (double)lastInt;
    
    int rightType = exp->right->accept(this);
    double rightVal = (rightType == 2) ? lastFloat : (double)lastInt;
    
    bool isFloatOperation = (leftType == 2 || rightType == 2);
    
    if (isFloatOperation) {
        switch (exp->op) {
            case PLUS_OP:
                lastFloat = leftVal + rightVal;
                lastType = 2;
                return 2;
            case MINUS_OP:
                lastFloat = leftVal - rightVal;
                lastType = 2;
                return 2;
            case MUL_OP:
                lastFloat = leftVal * rightVal;
                lastType = 2;
                return 2;
            case DIV_OP:
                if (rightVal == 0.0) {
                    throw runtime_error("División por cero");
                }
                lastFloat = leftVal / rightVal;
                lastType = 2;
                return 2;
            case LT_OP:
                lastInt = (leftVal < rightVal) ? 1 : 0;
                lastType = 3;
                return 3;
            // ... otros operadores de comparación
        }
    }
    // ... manejo de operaciones enteras
}
```

Esta implementación asegura que las operaciones mixtas (como `3.14 + 2`) se manejen correctamente, promoviendo el entero a flotante antes de realizar la operación. El manejo de errores incluye detección de división por cero específica para operaciones flotantes.

Las operaciones compuestas se implementaron extendiendo la clase `AssignStatement` para manejar operadores como `+=`, `-=`, `*=`, y `/=`:

```cpp
void EvalVisitor::visit(AssignStatement* stm) {
    if (stm->op == AssignStatement::PLUS_ASSIGN_OP) {
        // Obtener valor actual de la variable
        int currentType = getVariableType(stm->id);
        double currentVal = (currentType == 2) ? 
                           env.getFloat(stm->id) : 
                           (double)env.getInt(stm->id);
        
        // Evaluar lado derecho
        int rightType = stm->rhs->accept(this);
        double rightVal = (rightType == 2) ? lastFloat : (double)lastInt;
        
        // Realizar operación y asignar
        if (currentType == 2 || rightType == 2) {
            double result = currentVal + rightVal;
            env.setFloat(stm->id, result);
        } else {
            int result = (int)currentVal + (int)rightVal;
            env.setInt(stm->id, result);
        }
    }
    // ... otros operadores compuestos
}
```

### 3. Estructura de Visitors y Diseño

El diseño arquitectónico del compilador se basa en el patrón Visitor, que proporciona una separación clara entre la estructura de datos (AST) y las operaciones que se realizan sobre ella. Esta arquitectura ha demostrado ser especialmente valiosa para la integración de los nuevos tipos String y Float.

#### PrintVisitor

El `PrintVisitor` se encarga de generar una representación textual del AST que facilita la depuración y comprensión del análisis sintáctico. Para los nuevos tipos, se implementaron métodos específicos que manejan la visualización apropiada:

```cpp
int PrintVisitor::visit(StringExp* exp) {
    imprimirIndentacion();
    cout << "StringLiteral: \"" << exp->value << "\"" << endl;
    return 0;
}

int PrintVisitor::visit(DecimalExp* exp) {
    imprimirIndentacion();
    cout << "DecimalLiteral: " << exp->value << endl;
    return 0;
}

void PrintVisitor::imprimirIndentacion() {
    for (int i = 0; i < indent; i++) {
        cout << "  ";
    }
}
```

El sistema de indentación se implementó para mejorar la legibilidad de la salida, creando una estructura jerárquica visual que refleja la estructura del AST. Cada nivel de anidamiento se representa con dos espacios adicionales, facilitando la identificación de relaciones padre-hijo en el árbol.

#### EvalVisitor

El `EvalVisitor` implementa el intérprete del lenguaje, ejecutando directamente el código representado en el AST. Su diseño incluye un sistema sofisticado de manejo de tipos que permite operaciones polimórficas y conversiones automáticas:

```cpp
class EvalVisitor : public Visitor {
private:
    Environment env;
    int lastType;    // 1=Int, 2=Float, 3=Boolean, 5=String
    int lastInt;
    float lastFloat;
    string lastString;
    
public:
    int visit(StringExp* exp) override {
        lastString = exp->value;
        lastType = 5;
        return 5;
    }
    
    int visit(DecimalExp* exp) override {
        lastFloat = (float)exp->value;
        lastType = 2;
        return 2;
    }
};
```

El sistema de tipos utiliza variables de estado (`lastType`, `lastInt`, `lastFloat`, `lastString`) para mantener el resultado de la última evaluación. Este diseño permite que las operaciones accedan al valor y tipo del resultado sin necesidad de estructuras de datos complejas o boxing/unboxing explícito.

El `Environment` maneja el almacenamiento de variables con soporte para múltiples tipos:

```cpp
class Environment {
private:
    unordered_map<string, int> intVars;
    unordered_map<string, float> floatVars;
    unordered_map<string, string> stringVars;
    unordered_map<string, int> varTypes;
    
public:
    void setInt(const string& name, int value) {
        intVars[name] = value;
        varTypes[name] = 1;
    }
    
    void setFloat(const string& name, float value) {
        floatVars[name] = value;
        varTypes[name] = 2;
    }
    
    void setString(const string& name, const string& value) {
        stringVars[name] = value;
        varTypes[name] = 5;
    }
    
    int getType(const string& name) {
        return varTypes.count(name) ? varTypes[name] : 0;
    }
};
```

#### Diseño Arquitectónico y Extensibilidad

La arquitectura basada en el patrón Visitor proporciona varios beneficios fundamentales que fueron cruciales para la implementación exitosa de String y Float:

**Separación de Responsabilidades**: Cada visitor se enfoca en un aspecto específico del procesamiento. El `PrintVisitor` maneja únicamente la representación textual, mientras que el `EvalVisitor` se concentra en la ejecución. Esta separación permitió implementar el soporte para nuevos tipos sin afectar funcionalidades existentes.

**Extensibilidad**: La adición de nuevos tipos requirió únicamente la implementación de métodos `visit` específicos en cada visitor existente, sin modificar la estructura base del AST ni las interfaces existentes. Esto demuestra la robustez del diseño para futuras extensiones.

**Polimorfismo Dinámico**: El método `accept` en cada nodo del AST utiliza polimorfismo dinámico para invocar el método `visit` apropiado en cada visitor:

```cpp
int StringExp::accept(Visitor* visitor) {
    return visitor->visit(this);
}

int DecimalExp::accept(Visitor* visitor) {
    return visitor->visit(this);
}
```

Este diseño permite que el mismo AST sea procesado por múltiples visitors sin conocimiento explícito del tipo de visitor, facilitando la reutilización y el mantenimiento del código.

**Mantenibilidad**: Los cambios en la lógica de procesamiento de un tipo específico se localizan en los métodos correspondientes de cada visitor, minimizando el impacto de modificaciones y simplificando la depuración y el mantenimiento del sistema.

## Conclusión

La implementación exitosa de tipos String y Float en este compilador demuestra la flexibilidad del diseño arquitectónico empleado. El uso del patrón Visitor permite una gestión eficiente de los diferentes tipos de datos, manteniendo la claridad del código y facilitando futuras extensiones del lenguaje.
