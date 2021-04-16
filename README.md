```

LEXEUR:

struct token:
  - type // Type of token: cmd, pipe, redir
  - (void*) data can be a char* for filenames or a struct token* for args linked list
  - next // next elem in linked list


main_parse:

while {
  - lex_ifs
  - lex_token
  - lex_ifs
  - lex_sep
} // if loop until the line is end or ERROR


lex_ifs:

// return true if the given ascii is a space or a whitespace
return { is_white_space or is_space }


lex_token:

// Parse a command (data between pipes or just the data if there aren't pipes)
// If a command was found, loop until lex_jobs stop founding jobs (skipping spaces)
// NOTE: lex_jobs parse all the pipes and commands between the pipes
if {
  - lex_cmd == found
} == true -> while {
  - lex_ifs
  - it++
  - lex_jobs
}


lex_jobs:

- lex_pipe // just parse the char '|'
- lex_ifs // skip spaces
- lex_cmd // parse the data inter pipes


lex_cmd:

// Skip spaces
while {
  - lex_ifs
  - it++
}
// Check if lex_redir found a redirection and use it data, if not parse the params avoiding redirection check
if !lex_redir
  lex_param
 
 // New token holds lex_redir or lex_param data (data in general)
 CREATE NEW TOKEN AND ADD TO THE LINKED LIST -> token type is CDM
 
 
 
 lex_redir:
 
 - lex_redir_type // just check for ">" ">>" "<"
 - lex_param (parse filename) // fileame will be at ((struct token*)TOKEN_REDIR->data)->data

CREATE A NEW TOKEN -> token type is REDIR (3 types for redir are need)


lex_redir_type:

if ">" {
  it++
  if ">"{
    return ">>"
   else
    return ">"
} else if "<"{
  return "<"
} else {
  return NO_REDIR
}


lex_param:
 
 // Parse all params with or witout quotes
 while {
 - find params
 } do: {
 // No quotes ? lex_param_simple
 if !lex_param_quoted
    lex_param_simple
 }
 
  
 lex_param_simple:
 
 start = it;
 while {
 - !is_spetial // not a separator, pipe ...
 - it++
 }

CREATE A NEW TOKEN AND ADD IT TO THE (data*) of the token CMD or REDIR
(start ... it) data segment representing 1 parameter (argument)


lex_param_quoted:

// Test for double quotes, if no match check for single quotes
if !lex_param_dquote
  lex_param_squote

 
 lex_param_dquote:
 
 - if *it != '"'{
    return NO_MACTH
 }
 - it++ // skip dquote
 - start = it
 while {
 - *it != '"'
 } do: {
 - check_backslash // transform to the transformed value
 - it++
 }
 CREATE A NEW TOKEN AND ADD IT TO THE (data*) of the token CMD or REDIR
 (start ... it) data segment representing 1 parameter (argument)

lex_param_squote:

if *it != '\'' {
  return NO_MATCH
}
- it++ // skip squote
- start = it
while {
- *it != '\''
} do: {
  it++
}
 CREATE A NEW TOKEN AND ADD IT TO THE (data*) of the token CMD or REDIR
 (start ... it) data segment representing 1 parameter (argument)
 
 
 
 /*
 L'idée est de faire une liste chainée de tokens avec des differents types:
 - PIPE: type=PIPE, data=NULL
 - CMD: type=CMD, data=(struct token*) // une autre liste chainée pour les arguments
 - REDIR: type=REDIR_< ou REDIR_> ou REDIR_>>, data= Au choix ou bien un char* ou bien un maillon struct token* (represente le filename)
 
 Plus tard une fois que la liste chainée est faitte on peut facilement utiliser l'ordre des maillons pour savoir comment executer.
 A faire plus tard.
 
 Objectif, parser correctement toute sorte de commande et creeer des liste chainées.
 */
 ```

