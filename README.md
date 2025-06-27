# Compilador Kotlin - Implementación de Tipos String y Float

## Introducción

Este proyecto presenta la implementación de un compilador para un subconjunto del lenguaje Kotlin, con especial enfoque en la incorporación de tipos de datos **String** y **Float**. Si bien el compilador maneja estructuras de control tradicionales como bucles `for`, `while` y declaraciones condicionales, el núcleo de esta implementación se centra en la gestión avanzada de cadenas de texto y números en coma flotante.

## Gramática del Lenguaje

El compilador sigue la siguiente gramática EBNF:

# Gramática EBNF - Compilador Kotlin

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

#### Tipo String
- **StringExp**: Clase que representa literales de cadena en el AST
- **STRING_TYPE**: Token específico para el tipo String en el scanner
- **Manejo de cadenas**: Implementación de escape sequences y caracteres especiales

#### Tipo Float
- **DecimalExp**: Clase que representa números de coma flotante en el AST
- **DECIMAL**: Token para reconocer literales decimales con notación 'f'
- **Gestión de precisión**: Manejo de diferentes formatos de números flotantes

### 2. Operaciones Implementadas

#### Operaciones con String
- **Concatenación**: Uso del operador `+` para unir cadenas
- **Comparación**: Operadores de igualdad (`==`, `!=`) para strings
- **Asignación**: Soporte completo para asignación de variables String
- **Impresión**: Integración con `print()` y `println()`

#### Operaciones con Float
- **Operaciones aritméticas**: Suma, resta, multiplicación y división
- **Operaciones de comparación**: Todos los operadores relacionales (`<`, `<=`, `>`, `>=`, `==`, `!=`)
- **Conversiones de tipo**: Conversión automática entre Int y Float
- **Operaciones compuestas**: Asignación con operadores (`+=`, `-=`, `*=`, `/=`)

### 3. Estructura de Visitors y Diseño

#### PrintVisitor
- **Visualización de AST**: Impresión jerárquica del árbol sintáctico
- **Soporte para nuevos tipos**: Manejo específico de StringExp y DecimalExp
- **Indentación**: Sistema de indentación para mejorar la legibilidad

#### EvalVisitor
- **Evaluación de expresiones**: Interpretación directa de operaciones String y Float
- **Manejo de tipos**: Sistema de tipos dinámico con soporte para conversiones
- **Entorno de ejecución**: Gestión de variables y scope para los nuevos tipos
- **Operaciones mixtas**: Soporte para operaciones entre diferentes tipos de datos

#### Diseño Arquitectónico
El diseño sigue el patrón Visitor para separar la estructura del AST de las operaciones que se realizan sobre él. Esto permite:

- **Extensibilidad**: Fácil incorporación de nuevos tipos de datos
- **Separación de responsabilidades**: Cada visitor maneja un aspecto específico
- **Mantenibilidad**: Cambios en un visitor no afectan a otros
- **Reutilización**: El mismo AST puede ser procesado por múltiples visitors

## Conclusión

La implementación exitosa de tipos String y Float en este compilador demuestra la flexibilidad del diseño arquitectónico empleado. El uso del patrón Visitor permite una gestión eficiente de los diferentes tipos de datos, manteniendo la claridad del código y facilitando futuras extensiones del lenguaje.
