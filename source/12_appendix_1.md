# Appendix 1: Language Grammar {.language-grammar}

\begin{lstlisting}[caption=General rules,label={lst:general-grammar},language=BNF]
capitalized_identifier ::= [A-Z_][A-Za-z0-9_]*

identifier ::= [A-Za-z_][A-Za-z0-9_]*

string_literal ::= ''' [^']* '''

boolean_literal ::= true | false

integer_literal ::= digit+

digit ::= ['0'-'9']

variable_literal ::= identifier ('.' identifier)*

literal = string_literal | boolean_literal | integer_literal | variable_literal

type ::= 'String' | 'Int' | 'Boolean' | 'DateTime' | capitalized_identifier

parameter ::= identifier ':' type

connect_with_expr ::= 'connect' variable_literal 'with' variable_literal

permission ::= 'IsAuth' | 'OwnsRecord'

permissions_attribute ::= '\@permissions' '(' permission (',' permission)* ')'

http_method ::= 'get' | 'post' | 'put' | 'delete'

route_attribute ::= '\@route' '(' (http_method ',')? string_literal ')'

model_attribute ::= '\@model' '(' capitalized_identifier ')'

custom_entries ::= 'fn' ':' '[|' string_literal '|]' (',' 'imports' ':' '[|' string_literal '|]')?
\end{lstlisting}

**App Declaration Grammar**

\begin{lstlisting}[caption=App declaration rules,label={lst:grammar-app},language=BNF]
app_declaration ::= 'app' capitalized_identifier '{' app_entries '}'

app_entries ::= 'title' ':' string_literal (',' auth_entry)?

auth_entry ::= 'auth' ':' '{' auth_entries '}'

auth_entries ::= 'userModel' ':' capitalized_identifier ','
                 'idField' ':' identifier ','
                 'isOnlineField' ':' identifier ','
                 'lastActiveField' ':' identifier ','
                 'usernameField' ':' identifier ','
                 'passwordField' ':' identifier ','
                 'onSuccessRedirectTo' ':' string_literal ','
                 'onFailRedirectTo' ':' string_literal
\end{lstlisting}

**Data Models Grammar**

\begin{lstlisting}[caption=Model declaration rules,label={lst:model-grammar},language=BNF]
model_name ::= 'model' capitalized_identifier '{' model_fields '}'

model_fields ::= model_field (',' model_field)*

model_field ::= identifier type field_attributes*

field_attributes ::= '\@id' | '\@unique' | '\@default(value)' | '\@updatedAt'
\end{lstlisting}

**Queries Grammar**

\begin{lstlisting}[caption=Query declaration rules,label={lst:query-grammar},language=BNF]
query_declaration ::= query_attributes+ 'query' '<' query_type '>' identifier '{' query_entries '}'

query_attributes ::= permissions_attribute | model_attribute | route_attribute

query_entries = where_entry | search_entry | data_entry | custom_entries

query_type ::= 'Create' | 'FindUnique' | 'FindMany' | 'Update' | 'Delete' | 'Custom'

where_entry ::= identifier

search_entry ::= '[' identifier+ ']'

data_entry ::= 'fields' ':' '[' identifier+ ']' (',' relation_field_entry)?

relation_field_entry ::= 'relationFields' ':' '{' relation_field (',' relation_field)* '}'

relation_field ::= identifier ':' connect_with_expr
\end{lstlisting}

**XRA Grammar**

\begin{lstlisting}[caption=XRA rules,label={lst:xra-grammar},language=BNF]
html_tag ::= SET_OF_HTML_TAGS

xra_element_name ::= html_tag | capitalized_identifier

xra_element ::= self_closing_xra_element | xra_element_opening xra_children xra_element_closing | xra_for_expression | xra_if_expression

xra_fragment ::= '<>' xra_element+ '</>'

xra_children ::= xra_element | xra_literal_expression

xra_element_opening ::= '<' xra_element_name xra_element_attribute* '>'

xra_element_closing ::= '</' xra_element_name '>'

self_closing_xra_element ::= '<' xra_element_name xra_element_attribute* '/>'

xra_element_attribute ::= identifier '=' xra_literal_expression | string_literal | boolean_literal | integer_literal

xra_literal_expression ::= '{' literal '}'

xra_for_expression ::=
  '[%' 'for' identifier 'in' identifier '%]'
    xra_element | xra_fragment
  '[% endfor %]'

xra_if_expression ::=
  '[%' 'if' condition '%]'
    xra_element | xra_fragment
  '[%' 'endif' '%]'

condition ::= literal logical_operation literal

logical_operation ::= '==' | '<' | '>' | '<='  | '>='

render_expression ::= xra_element+ | xra_fragment+
\end{lstlisting}

**Component Grammar**

\begin{lstlisting}[caption=Component declaration rules,label={lst:component-grammar},language=BNF]
component_declaration ::= 'component' '<' component_type '>' capitalized_identifier ('(' parameter* ')')? { component_body }

component_body ::= render_expression | fetch_component_entries | action_form_component_entries | action_button_component_entries | custom_entries

component_type ::= 'Create' | 'FindUnique' | 'FindMany' | 'Update' | 'Delete' | 'Custom' | 'SignupForm' | 'LoginForm' | 'LogoutButton'

find_component_entries ::=
  'findQuery' ':' find_query ','
  'onError' ':' render_expression ','
  'onLoading' ':' render_expression ','
  'onSuccess' ':' render_expression

find_query ::= identifier '(' ')'

find_query_entries ::= '{'
  'where' ':' '{' where_or_search_field+ '}'
  | 'search' ':' '{' where_or_search_field+ '}'
'}'

where_or_search_field ::=  identifier ':' literal

action_form_component_entries ::=
  'actionQuery' ':' identifier '()' ','
  'formInputs' ':' '{' form_input_entry '}' ','
  'formButton' ':' '{' form_button_entry '}'
  (',' 'globalStyle' ':' '{' global_style_entry '}')?

action_button_component_entries ::=
  'actionQuery' ':' identifier '()' ','
  form_name_entry
  (',' form_style_entry)?

form_input_type ::= 'TextInput' | 'EmailInput' | 'PasswordInput' | 'NumberInput' | 'NumberInput' | 'CheckboxInput' | 'DateTimeInput' | 'DateInput'

global_style_entry ::= 'formContainer' ':' string_literal
                    | 'inputContainer' ':' string_literal
                    | 'input' ':' string_literal
                    | 'inputError' ':' string_literal
                    |  'inputLabel' ':' string_literal

form_input_entry ::= identifier ':' '{' form_input_entry_options '}'

form_input_entry_options ::=
'input' ':'
  '{' form_input_type_entry
      (',' form_input_default_value
      ',' form_input_placeholder
      ',' form_input_visibility
      ',' form_style_entry)? '}'
    (',' 'label' ':'
      '{' form_name_entry (',' form_style_entry )? '}')?
    (',' form_style_entry)?

form_input_type_entry ::= 'type' ':' form_input_type

form_input_default_value ::= 'defaultValue' ':' literal | connect_with

form_input_visibility ::= 'isVisible' ':' boolean_literal

form_input_placeholder ::= 'placeholder' ':' string_literal

form_button_entry ::= '{' form_name_entry (',' form_style_entry)? '}'

form_name_entry ::= 'name' ':' string_literal

form_style_entry ::= 'style' ':' string_literal
\end{lstlisting}

**Page Declaration**

\begin{lstlisting}[caption=Page declaration rules,label={lst:page-grammar},language=BNF]
page_declaration ::= page_attributes+ 'page' capitalized_identifier ('(' parameter* ')')? { render_expression }

page_attributes ::= permissions_attribute | route_attribute
\end{lstlisting}
