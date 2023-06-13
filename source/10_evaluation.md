\chapter{Evaluation}
\label{chap:eval}

We will demonstrate the DSL's effectiveness in building fully-functional web apps with varying requirements and features. We will then conduct user experiments to evaluate the learning curve, syntax, and developer experience of the DSL among developers with varying levels of experience. The data collected will include the time taken to complete each task, ease of understanding the DSL syntax, and any challenges encountered. Finally, we aim to identify and address any limitations or issues of the DSL to inform future development and improvements.

## Evaluation through Application Development

The DSL aims to streamline the web app development process, minimize errors, and reduce the time and effort required. To showcase the language's effectiveness, we have developed 3 web apps that exemplify its capabilities. Each app highlights distinct features of the DSL, illustrating how it can rapidly produce functional web apps. The source code for all the 3 applications mentioned below can be found in Appendix [3](#appendix-3-applications-source-code).

**Todo App**

The Todo app is designed for creating, updating, and deleting tasks while incorporating an authentication feature to restrict unauthorized access and actions. Its interactive interface provides real-time updates, eliminating the need for page refreshes. One of the significant benefits of using the DSL to build the Todo app is its capability to implement the permissions feature easily. By using the `@permissions` attribute, it was effortless to specify that only authorized users could access their tasks while restricting access to tasks created by other users. As shown in Figure @fig:todo-app-ui, there are two users using the app and each of them can only see their own tasks. This streamlined approach simplifies the development process and ensures secure and efficient functionality.

\begin{figure}[H]
  \centering
  \includegraphics[width=\textwidth]{source/figures/todo-app.png}
  \caption{Todo app user interface.}
  \label{fig:todo-app-ui}
\end{figure}

**Chatroom**

The Chatroom app provides users with the ability to create accounts and engage in conversations with other users. It also includes a user list that displays all active users along with their online/offline status. The app leverages real-time updates, eliminating the need for manual refreshing. These updates are facilitated by a server that pushes events to clients using SSE, with the clients automatically updating their local state in real-time. The UI for this application is shown in Figure @fig:chatroom-ui, where we have two users Alice and Bob messaging each other, and the state of the client app is updated in real-time. Whenever any of them sends a message the message will be loaded for the other user automatically without refreshing the page, as well as the status of each user will be updated. This app can have many users chatting in the same chatroom and the app will update the state of all the clients in real-time.

\begin{figure}[H]
  \centering
  \includegraphics[width=\textwidth]{source/figures/chatroom.png}
  \caption{Chatroom user interface.}
  \label{fig:chatroom-ui}
\end{figure}

**Kanban Board**

The Kanban board app is a collaborative app that allows different users to use a Kanban board to organize their tasks together. The app implements three columns: To do, In progress, and Done. Users can create new tasks and they can drag and drop any task from one column to another to update its state. Like the previously mentioned apps, all updates in this app happen in real-time. Moreover, this app shows that developers can interop with TypeScript and use libraries available in the JavaScript/TypeScript ecosystem. To implement this app, we used a library called react-beautiful-dnd[@dnd-ref], which allowed us to create the drag-and-drop feature in the app easily. As shown in Figure @fig:kanban-ui users can create tasks and drag them between the different columns. Like the chatroom app, the state of this app is updated between all the clients logged in automatically in real-time.

\begin{figure}[H]
  \centering
  \includegraphics[width=\textwidth]{source/figures/kanban.png}
  \caption{Kanban board user interface.}
  \label{fig:kanban-ui}
\end{figure}

## Evaluation on Requirements

The DSL simplifies web development by abstracting many low-level details, allowing developers to focus on high-level functionality and features. We evaluated the effectiveness of the DSL by comparing the lines of code required to build each app using the DSL versus using pure NodeJS and ReactJS. Table @tbl:lines-of-code shows that developers can create the Todo app and Chatroom app with just 15% of the lines of code required for NodeJS and ReactJS. While apps that require TypeScript, like the Kanban Board app, require slightly more lines of code in the DSL, the total is still much less than the lines of code required for pure NodeJS/ReactJS development.

| App            | DSL            | NodeJS/ReactJS |
| -------------- | -------------- | -------------- |
| Todo App       | 233            | 1547           |
| Chatroom       | 258            | 1580           |
| Kanban Board   | 405            | 1669           |

Table: Lines of code required to implement each app in the DSL and NodeJS/ReactJS including empty lines and excluding build configuration files. {#tbl:lines-of-code}

The DSL has been shown to meet all the requirements outlined in Chapter \ref{chap:analysis-and-specs} through the development of the three discussed apps.

- **R1. Support multitier programming**: We were able to build all three apps without the need for tiers because the compiler was responsible for tier splitting by leveraging the different declarations in the DSL. This allowed us to solve the impedance mismatch problem that is commonly encountered in web development. Additionally, the DSL's static-typing feature made it possible for the compiler to catch errors at compile time, reducing the risk of encountering runtime errors.

- **R2. Allow developing full-stack apps declaratively**: The use of declarations allowed us to abstract away low-level details, providing developers with high-level declarations to use instead. As a result, building all three apps in the DSL was significantly easier compared to using pure NodeJS and ReactJS, as shown in Table \ref{tbl:lines-of-code}. Also, using the different declarations in the DSL, the compiler was able to split the code into different tiers at compile time.

- **R3. Compile to human-readable and type-safe TypeScript code using existing tech stacks**: Using the generated IR and utilizing the pre-written templates, we managed to compile to a NodeJS server and a ReactJS client, producing human-readable and type-safe TypeScript code. As we discussed in Chapter \ref{chap:impl-details}.

- **R4. Real-time by default**: The Chatroom and Kanban Board apps both feature real-time updates, with the client state syncing between all connected clients. This was made possible through the use of SSE, which is explained in detail in Chapter \ref{chap:impl-details}.

- **R5. Allow interoperability with TypeScript**: The DSL provides developers with the ability to write inline TypeScript code using `Custom` queries and components. This feature allows for greater flexibility in application development and customization. Moreover, specifying JavaScript/TypeScript packages in the app configuration enables easy interoperability with TypeScript, as demonstrated in the Kanban Board app.

## User Evaluation

The online experiment conducted to evaluate the learning curve, syntax, and developer experience of the DSL was divided into two parts, each assessing different aspects of the participants' experience. In the first part, participants were presented with a working code snippet without any prior instruction on the DSL, aimed at evaluating their understanding of the code and assessing its comprehensibility. Each question in this section was divided into two parts: First, participants were asked to rate their understanding of the code snippet on a scale from 1 to 5, where a higher number indicated a better understanding of the code snippet. Second, they were asked to explain what they understood from the code snippet. This approach effectively measured the initial learning curve and syntax of the language.

The second part aimed to assess the developer experience by providing participants with a code snippet containing a bug and the corresponding compiler error message. Participants were then required to explain the error in the code and suggest a solution. The experiment's design ensured a diverse population of participants by asking about their programming and web development experience before the experiment.

After the experiment was completed, participants were allowed to reflect on their overall experience with the DSL and provide feedback on the language aspects that they found most appealing or confusing. This approach allowed us to gain valuable insights into the effectiveness of the DSL and identify areas that required improvement.

Of the 9 participants who took part in the experiment, most were experienced programmers, although not all had experience with web development. Impressively, 5 out of the 9 participants were able to explain all of the presented code snippets correctly, while the remaining 4 participants correctly answered an average of 8.75 questions. These results suggest that the language syntax is easy to comprehend and that the compiler messages were effective in identifying errors.

It is notable that, on average, it took the participants 33 minutes to solve all the questions, indicating that they were able to grasp the language's syntax quickly simply by reading it without any prior exposure to DSL. This highlights the language's intuitive nature and demonstrates the participants' ability to quickly comprehend the language's syntax.

The use of the `@` annotation to declare attributes were found to be confusing by many participants. Similarly, the `<>` notation used to specify the types of queries and UI components was found to be confusing by several participants, as it is commonly used to define type parameters in many programming languages.

A few participants had difficulty understanding the process of defining one-to-many relations, particularly in determining which parameter passed to the `@relation` attribute represents the foreign key. The `connect` notation used to connect records also proved challenging for some participants.

Despite encountering some confusing notations and not having received any training or documentation on the DSL, the majority of the participants were able to answer all questions correctly. This highlights the DSL's accessibility and user-friendly nature. Almost all of the participants found the syntax of the language easy to interpret, while the compiler messages were found to help identify and correct errors in the code. For more information about the questions asked and the data collected from the experiment, please refer to Appendix [4](#appendix-4-user-experiment).

## Issues

In this section, we will explain some issues related to the DSL. These issues include those related to the architecture of the compiler as well as those related to the application that the language compiles.

**Problems with REST APIs**

REST APIs present several challenges that can impact their performance and reliability. One common issue is returning excessive data that the client does not need, leading to large data responses and increased latency. Additionally, REST APIs may struggle to handle high levels of traffic, which can result in slow response times and even system crashes. To address this issue, it may be necessary to increase the number of servers running the API and implement efficient load-balancing and caching strategies to ensure optimal performance and reliability.

**Problems with SPAs**

One of the most significant issues with SPAs is their impact on search engine optimization (SEO). Search engines like Google rely heavily on content and links to index and rank websites. However, since SPAs only load a single HTML file and dynamically update the content using JavaScript, search engine crawlers have a difficult time indexing the content of the page[@seo-problem-ref]. This can result in a lower search engine ranking, making it harder for users to find the website.

Although it is a significant problem in SPAs, there are techniques to solve it, such as server-side rendering (SSR) and static site generation (SSG). In SSR, the server processes and renders the webpage before sending it to the client. When a user requests a page, the server generates the HTML, CSS, and JavaScript required for the page and sends it to the client. This approach means that the browser receives a fully rendered page that is ready to be displayed, reducing the amount of work the browser needs to do and improving the page load time. On the other hand, SSG generates the HTML, CSS, and JavaScript for a website at build time, rather than at runtime. The resulting files are then hosted on a web server and served to users as static pages. This approach allows for faster load times since the pages are pre-rendered and do not require any server processing when a user requests them. Additionally, it improves SEO, since search engines can easily crawl and index static pages. Therefore, sites that use SSR or SSG have the potential to rank higher in search engine results.

**Cannot Compile to All Existing Framework**

Generally, the AppSpecs are designed to be framework-agnostic, but some parts of it are generated specifically to work with certain frameworks and libraries. For instance, UI structures are translated to JSX[@jsx-ref], an XML-like language used by various UI frameworks such as ReactJS, Preact, SolidJS, and Qwik. However, not all UI frameworks use JSX; Angular and Svelte, for instance, use their own template syntax. As a result, the compiler's IR can only be utilized to generate code for frameworks that support JSX.

**Unsafe TypeScript Interoperability**

The DSL cannot currently check the inline TypeScript code and instead treats it as a string. This can lead to various issues, especially if the data returned by the inline code does not match the data specified in the query signature. Such inconsistencies can result in runtime errors and cause significant problems. Therefore, it is essential to exercise caution when working with inline TypeScript code in the DSL and ensure that the returned data aligns correctly with the expected data type.
