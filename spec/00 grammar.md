# Grammar

The grammar of the language in Extended Backus-Naur Form (EBNF).

    import           ::= [ "from" ( id | string) ] "import" ( id | string ) [ as ] { "," as }
    as               ::= id { "." id } "as" id

    body             ::= id [ "[" id_maybe_type { "," id_maybe_type } "]" ] [ "isa" id { "," id } ] newline { newline }

    type             ::= "type" type ( class-body | "isa" type [ conditions ] )
    script           ::= statements
    module           ::= ( "stateless" | "stateful" ) body | script | type
    file             ::= import | module { newline { newline } }
    
    id               ::= "self" | ( letter | "_" ) { character }
    id_or_call       ::= id [ ( [ "." ] id | [ "::" id ] ) ( tuple | expression ) ]

    generics         ::= "[" id { "," id } "]"
    type             ::= ( id [ generics ] | type-tuple ) [ "->" type ]
    type-tuple       ::= "(" [ type { "," type } ] ")"
    id-maybe-type    ::= id [ ":" type ]
    
    conditions       ::= "when" ( newline indent { condition } dedent | condition )
    condition        ::= expression [ "else" expression ]
    
    block            ::= indent statements dedent
    statements       ::= { expr-or-stmt { newline } }
    
    expr-or-stmt     ::= statement | expression [ ( raises | handle ) ]
    statement        ::= ( "print" | "println" ) expression
                      | control-flow-stmt
                      | definition
                      | type-def
                      | "retry"
    expression       ::= "(" expression ")" 
                      | expression ( ".." | "..=" ) expression
                      | expression "?or" expression
                      | "return" [ expression ]
                      | expression "as" id 
                      | control-flow-expr 
                      | newline block
                      | reassignment
                      | collection
                      | key-value
                      | operation
                      | anon-fun
                      | call
                      | "_"
                     
    reassignment     ::= expression "<-" expression
    anon-fun         ::= "\" id-maybe-type { "," id-maybe-type } "=>" expression
    call             ::= expression [ [ ( "." | "?." ) ] id ] ( tuple | expression )
    
    raises           ::= "raises" generics
    handle           ::= "handle" newline match-cases
    
    collection       ::= tuple | set | list | map
    tuple            ::= "(" zero-or-more-expr ")"
    set              ::= "{" zero-or-more-expr "}" | set-builder
    set-builder      ::= "{" expression "|" expression { "," expression } "}"
    list             ::= "[" zero-or-more-expr "]" | list-builder
    list-builder     ::= "[" expression "|" expression { "," expression } "]"
    list-head        ::= id "::" expression
    
    zero-or-more-expr::= [ zero-or-more-expr ]
    one-or-more-expr ::= expression { "," expression }
    
    definition       ::= "def" ( [ "private" ] ( variable-def | fun-def ) | operator-def )

    variable-def     ::= [ "mut" ] ( id-maybe-type | collection ) [ "ofmut" ] [ "<-" expression ] [ forward ]
    operator-def     ::= overridable-op [ "(" [ id-maybe-type ] ")" ] ":" type [ "->" expression ]
    fun-def          ::= id fun-args [ ":" type ] [ raises ] [ "=>" expression ]
    fun-args         ::= "(" [ fun-arg ] { "," fun-arg } ")"
    fun-arg          ::= [ "vararg" ] ( id-maybe-type | literal ) [ "<-" expression ]
    forward          ::= "forward" id { "," id }
    
    operation        ::= relation [ ( equality | instance-eq | binary-logic ) relation ]
    relation         ::= arithmetic [ comparison relation ]
    arithmetic       ::= term [ additive arithmetic ]
    term             ::= inner-term [ multiclative term ]
    inner-term       ::= factor [ power inner-term ]
    factor           ::= [ unary ] ( literal | id_or_call | expression )
    
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
    real             ::= integer "." integer | "." integer | integer "."
    integer          ::= { digit }
    e-notation       ::= ( integer | real ) ( "e" | "E" ) [ "-" ] integer
    boolean          ::= "True" | "False"
    string           ::= """ { character } """
                                     
    control-flow-expr::= if | match
    if               ::= "if" one-or-more-expr "then" expr-or-stmt [ "else" expr-or-stmt ]
    match            ::= "match" one-or-more-expr "with" newline match-cases
    match-cases      ::= indent { match-case { newline } } dedent
    match-case       ::= expression "=>" expr-or-stmt
    
    control-flow-stmt::= while | foreach | "break" | "continue"
    while            ::= "while" one-or-more-expr "do" expr-or-stmt
    foreach          ::= "foreach" one-or-more-expr "in" expression "do" expr-or-stmt
    
    newline          ::= \n | \r\n
    comment          ::= "#" { character }

An `expression` is used in a situation where an expression is required. 
However we cannot always know in advance whether this is the case, e.g. when it is a function call. 
In This should be verified by the type checker. 
An `expr-or-stmt` may be used when it does not matter whether something is an expression or statement, such as the body of a loop.
              
We do not systematically desugar multiple 
delimited by commas, or a single expression, to tuples, as is the case in Python.
This prevents ambiguity in the grammar as specified above, and also prevents confusing situations such as `(0)` and `0` being equal.
Instead, we only do this in specific contexts, such as in the conditional of control flows.
