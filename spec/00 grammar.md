# Grammar

The grammar of the language in Extended Backus-Naur Form (EBNF).

    import           ::= "from" id [ ( "use" { id { "," id } | "useall" ) ] [ "as" id ]
    class-body       ::= id [ "[" id_maybe_type { "," id_maybe_type } "]" ] [ "isa" id { "," id } ] newline { newline }
                         indent { definition newline { newline } } dedent
    type             ::= "type" class-body
    util             ::= "util" class-body
    class            ::= "class" class-body
    script           ::= statements
    module           ::= type | util | class | script
    file             ::= import | module { newline { newline } }
    
    id               ::= "self" | ( letter | "_" ) { ( letter | number | "_" ) }
    generics         ::= "[" id { "," id } "]"
    type             ::= id [ generic ] | type-tuple [ "->" type ]
    type-tuple       ::= "(" [ type { "," type } ] ")"
    type-def         ::= "type" id "isa" type [ conditions ]
    id-maybe-type    ::= id [ ":" type ]
    
    conditions       ::= "when" ( newline indent { condition } dedent | condition )
    condition        ::= expression [ "else" expression ]
    
    block            ::= indent statements dedent
    statements       ::= { expr-or-stmt { newline } }
    
    expr-or-stmt     ::= statement | expression
    statement        ::= ( "print" | "println" ) expression 
                      | statement ( raises | handle )
                      | control-flow-stmt
                      | definition
                      | type-def
                      | "retry"
    expression       ::= "(" expression ")" 
                      | newline block
                      | expression ( ".." | "..=" ) expression
                      | anon-fun
                      | expression ( raises | handle )
                      | "return" [ expression ]
                      | expression "?or" expression
                      | expression "as" id 
                      | control-flow-expr 
                      | call
                      | reassignment
                      | collection
                      | key-value
                      | operation
                      | "_"
                     
    reassignment     ::= expression "<-" expression
    anon-fun         ::= expression "->" expression
    call             ::= id ( [ "." ] id | [ "::" id ] ) ( tuple | expression )
    
    raises           ::= "raises" generics
    handle           ::= "handle" "when" newline when-cases
    
    collection       ::= tuple | set | list | map
    tuple            ::= "(" zero-or-more-expr ")"
    set              ::= "{" zero-or-more-expr "}" | set-builder
    set-builder      ::= "{" expression "|" expression { "," expression } "}"
    list             ::= "[" zero-or-more-expr "]" | list-builder
    list-builder     ::= "[" expression "|" expression { "," expression } "]"
    zero-or-more-expr::= [ ( expression { "," expression } ]
    
    definition       ::= "def" ( [ "private" ] ( variable-def | fun-def ) | operator-def | constructor )
    variable-def     ::= [ "mut" ] ( id-maybe-type | collection ) [ "ofmut" ] [ "<-" expression ] [ forward ]
    operator-def     ::= overridable-op [ "(" [ id-maybe-type ] ")" ] ":" type [ "->" expression ]
    fun-def          ::= id fun-args [ ":" type ] [ raises ] [ "->" expression ]
    fun-args         ::= "(" [ fun-arg ] { "," fun-arg } ")"
    fun-arg          ::= [ "vararg" ] ( id-maybe-type | literal ) [ "<-" expression ]
    forward          ::= "forward" id { "," id }

    constructor      ::= "init" constructor-args [ "<-" expr-or-stmt ]
    constructor-args ::= "(" [ constructor-arg { "," constructor-arg } ] ")"
    constructor-arg  ::= [ "vararg" ] id-maybe-type
    
    operation        ::= relation | relation ( equality | instance-eq | binary-logic ) relation
    relation         ::= arithmetic [ comparison relation ]
    arithmetic       ::= term [ additive arithmetic ]
    term             ::= inner-term [ multiclative term ]
    inner-term       ::= factor [ power inner-term ]
    factor           ::= [ unary ] ( literal | id | expression )
    
    overrideable-op  ::= additive | "sqrt" | multiplicative | power | "eq" | "<" | ">"
    unary            ::= "not" | "sqrt" | additive 
    additive         ::= "+" | "-"
    multiplicative   ::= "*" | "/"
    power            ::= "^" | "mod"
    instance-eq      ::= "is" | "isnt" | "isa" | "isnta"
    equality         ::= "eq" | "neq"
    comparison       ::= "<=" | ">=" | "<" | ">"
    binary-logic     ::= "and" | "or"
    
    literal          ::= number | boolean | string
    number           ::= real | integer | e-notation
    real             ::= digit "." { digit }
    integer          ::= { digit }
    e-notation       ::= ( integer | real ) ( "e" | "E" ) [ "-" ] integer
    boolean          ::= "true" | "false"
    string           ::= """ { character } """
                                     
    control-flow-expr::= if | from | when
    if               ::= "if" expression "=>" expr-or-stmt [ "else" expr-or-stmt ]
    when             ::= "when" expression newline when-cases
    when-cases       ::= indent { when-case { newline } } dedent
    when-case        ::= expression "=>" expr-or-stmt
    
    control-flow-stmt::= while | foreach | "break" | "continue"
    while            ::= "while" expression "=>" expr-or-stmt
    foreach          ::= "foreach" expression { "," expression } "in" expression "=>" expr-or-stmt
    
    newline          ::= \n | \r\n
    comment          ::= "#" { character }

The language uses indentation to denote code blocks. The indentation amount can't be described in the grammar directly, 
but it does adhere to the following rules:

* Every new expression or statement in a block must be preceded by n + 1 `indent`'s, where n is the amount of 
  `indent`'s before the block
* The same holds for every new `when-case` in a `when`

A `expression` is used in a situation where an expression is required. However we cannot always know in advance whether
this is the case, e.g. when it is a function call. In This should be verified by the type checker.
`expr-or-stmt` may be used when it does not matter whether something is an expression or statement, such as the body of
a loop.
               