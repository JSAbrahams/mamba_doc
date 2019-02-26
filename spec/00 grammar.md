# Grammar

The grammar of the language in Extended Backus-Naur Form (EBNF):

    import           ::= [ "from" ( id | string) ] "import" ( id | string ) [ as ] { "," as }
    as               ::= id { "." id } "as" id
    
    body             ::= id [ "[" id_maybe_type { "," id_maybe_type } "]" ] [ "isa" id { "," id } ] newline { newline }
                         indent { definition newline { newline } } dedent
    script           ::= statements
    module           ::= "def" ( "type" | "stateless" | "stateful" ) body | script
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
                      | collection-head
                      | call
                      | reassignment
                      | collection
                      | key-value
                      | operation
                      | "_"
                     
    reassignment     ::= expression "<-" expression
    anon-fun         ::= expression "->" expression
    (* methods or functions cannot be called using postfix notation if they take a tuple as argument *)
    call             ::= id [ [ "." ] id ] ( tuple | expression )
    
    raises           ::= "raises" generics
    handle           ::= "handle" "when" newline when-cases
    
    collection       ::= tuple | set | list | map
    tuple            ::= "(" zero-or-more-expr ")"
    set              ::= "{" zero-or-more-expr "}" | set-builder
    set-builder      ::= "{" expression "|" expression { "," expression } "}"
    list             ::= "[" zero-or-more-expr "]" | list-builder | list-head
    list-head        ::= id "::" expression
    list-builder     ::= "[" expression "|" expression { "," expression } "]"
    zero-or-more-expr::= [ expression { "," expression } ]
    
    definition       ::= "def" [ "private" ] ( variable-def | fun-def | operator-def )
    
    variable-def     ::= [ "mut" ] ( var-def | col-def )
    var-def          ::= id-maybe-type [ "<-" expression ] [ forward ]
    col-def          ::= collection [ "ofmut" ] [ "<-" expression ]
    
    operator-def     ::= overridable-op [ "(" [ id-maybe-type ] ")" ] ":" type [ "->" expression ]
    fun-def          ::= id fun-args [ ":" type ] [ raises ] [ "->" expression ]
    fun-args         ::= "(" [ fun-arg ] { "," fun-arg } ")"
    fun-arg          ::= id-maybe-type [ ( "<-" expression | "*" ) ] | literal
    
    forward          ::= "forward" id { "," id }
    
    operation        ::= relation [ ( equality | instance-eq | binary-logic ) relation ]
    relation         ::= arithmetic [ comparison relation ]
    arithmetic       ::= term [ additive arithmetic ]
    term             ::= inner-term [ multiclative term ]
    inner-term       ::= factor [ power inner-term ]
    factor           ::= [ unary ] ( literal | id ) | unary expression
    
    overrideable-op  ::= additive | "sqrt" | multiplicative | power | "=" | "<" | ">"
    unary            ::= "not" | "sqrt" | additive 
    additive         ::= "+" | "-"
    multiplicative   ::= "*" | "/"
    power            ::= "^" | "mod"
    instance-eq      ::= "is" | "isnt" | "isa" | "isnta"
    equality         ::= "=" | "/="
    comparison       ::= "<=" | ">=" | "<" | ">"
    binary-logic     ::= "and" | "or"
    
    literal          ::= number | boolean | string
    number           ::= real | integer | e-notation
    real             ::= digit "." { digit }
    integer          ::= { digit }
    e-notation       ::= ( integer | real ) ( "e" | "E" ) [ "-" ] integer
    boolean          ::= "True" | "False"
    string           ::= "'" { character } "'"
                                     
    control-flow-expr::= if | match
    if               ::= "if" expression { "," expression } "=>" expr-or-stmt [ "else" expr-or-stmt ]
    match            ::= "match" expression { "," expression } "with" newline when-cases
    match-cases      ::= indent { when-case { newline } } dedent
    match-case       ::= expression { "," expression }  "=>" expr-or-stmt
    
    control-flow-stmt::= while | foreach | "break" | "continue"
    while            ::= "while" expression { "," expression } "=>" expr-or-stmt
    foreach          ::= "foreach" expression { "," expression } "in" expression "=>" expr-or-stmt
    
    newline          ::= \n | \r\n
    comment          ::= "#" { character }

An `expression` is used in a situation where an expression is required. 
However we cannot always know in advance whether this is the case, e.g. when it is a function call. 
In This should be verified by the type checker. 
An `expr-or-stmt` may be used when it does not matter whether something is an expression or statement, such as the body of a loop.
              
We do not systematically desugar multiple expressions delimited by commas, as is the case in python, to tuples.
This prevents ambiguity in the grammar as specified above, and also prevents confusing situations such as `(0)` and `0` being equal.
Instead, we only do this in specific contexts, such as in the conditional of control flows.
