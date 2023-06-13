# Abstract {.unnumbered}

Multitier programming enables the development of the typical 3-tiers (database, server, and client) of a web application in a single compilation unit using a multitier language. This approach makes developing web applications easier than the current approach, which requires combining multiple tools and programming languages. There have been many multitier programming languages proposed in the literature, but they have failed to gain the attention of the community because they usually lack support for the tools required to build web applications.

We advocate that multitier programming languages should be interoperable with existing general-purpose languages, enabling developers to benefit from the rich ecosystem of these languages. The proposed language is designed to address the impedance mismatch problem that can happen in the 3-tier architecture as well as reduce the amount of boilerplate code required in web app development. It offers a set of declarations that can be used to develop all parts of web applications and abstracts the low-level details of web development. Additionally, it is designed to be interoperable with TypeScript to address the problem of adoption that faced previous multitier languages. This enables developers to benefit from the rich ecosystem of JavaScript/TypeScript. The language also compiles to human-readable code using existing tech stacks, giving developers the freedom to take the compiled code and further develop their apps when the language becomes limiting.

The language is evaluated through a series of experiments that measure its effectiveness in reducing development effort, eliminating the impedance mismatch problem, and reducing the learning curve. The results indicate that the language can significantly reduce the amount of code required to build a web app while improving the quality of the code and that it is easy to learn.

\pagenumbering{roman}
\setcounter{page}{1}
\newpage
