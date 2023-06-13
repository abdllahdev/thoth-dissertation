# Appendix 2: Source Code {.source-code}

**Prerequisites**

Before you can build and run this code, you will need to have the following installed on your system:

1- OCaml = v4

2- Dune = v3.6

3- NodeJS = v16

4- PostgreSQL = v14.7

**Building**

To build the compiler, please follow the instructions below:

\begin{lstlisting}[language=Thoth]
dune build bin/main.exe
\end{lstlisting}

**Running**
To run the compile, please follow the instructions below:

Any app can be compiled using the following command.

\begin{lstlisting}[language=Thoth]
dune exec -- bin/main.exe PATH_TO_APP DATABASE_NAME
\end{lstlisting}

*Command Line Options*

The following command line options are available for this project:

- --output_dir: The path to the directory that will contain the compiled code. Default value: ".out"

- --server_port: The port used to run the server on. Default value = 4000

To use these options, simply include them when compiling the app from the command line, as follows:

\begin{lstlisting}[language=Thoth]
dune exec -- bin/main.exe PATH_TO_APP DATABASE_NAME --output_dir=PATH_TO_DIR --server_port=PORT_NUMBER
\end{lstlisting}

The compiler creates two different directories, one for the server and another for the client. To run the compiled app, first, ensure that the PostgreSQL server is running on the machine. Next, navigate to the server directory and use the following command to create the database:

\begin{lstlisting}[language=Thoth]
yarn prisma migrate dev
\end{lstlisting}

After that, you can run the server using the following command:

\begin{lstlisting}[language=Thoth]
yarn dev
\end{lstlisting}

To run the client, navigate to the client directory and use the same command that was used to run the server.

**Folder Structure**

\begin{lstlisting}[language=Thoth]
/main_dir
  /.vscode // contains settings for vscode
  /bin // contains the executable file for the compiler
  /examples // contains examples for apps built using the DSL
  /language_tools // contains the code for the VSCode syntax highlighting extension
  /src // contains the main code for the compiler
  |  /analyzer // contains the code for the lexer, parser, and the type checker
  |  |  /ast // the package responsible for maintaining and formatting the AST
  |  |  /error_handler // the package responsible for creating error messages
  |  |  /parsing // the package responsible for lexing and parsing the code
  |  |  /type_checker // the package used for type checking
  |  |  |  checker.ml // the main checker file, it calls the symbol table builder and the checker
  |  |  |  environment.ml // the file that defines the symbol table data structure
  |  |  |  model_check.ml // implements the code responsible for checking data models
  |  |  |  query_checker.ml // implements the code responsible for checking queries models
  |  |  |  type_decl_checker.ml // implements the code responsible for checking the type declaration
  |  |  |  xra_checker.ml // implements the code responsible for checking the UI declarations
  |  /generator // the package used to generate the code
  |  /specs // the package used to generate the IR
  /templates // contains the templates for the db, client, server
  |  /client
  |  /db
  |  /server
\end{lstlisting}
