# Building Your First Extension

## Build an Extension as a GitHub App

### Building in a User Account

In this workshop you’ll build your first GitHub Copilot Extension as a GitHub App. You’ll create an ASP.NET Core minimal web API using C#.


1. Using your web browser, log into your GitHub account <b>with your personal account</b>. It could be easier to use a private session to avoid mixing it with your professional account.

2. Logout your professional account and login with your personal one also in VSCode. (you could use VSCode insiders if you prefer but it isn't required)

3. Let's choose the extensionname for the next steps. To be sure that it will be unique please choose something like [yourgithubusername]-workshop-6

3. You should be in a folder like /workspace-6. Create a new folder called src by typing the following and pressing Enter:

    ```bash
    mkdir src
    ```

4. Next change into the directory by typing the following and pressing Enter:

    ```bash
    cd src
    ```

5. In the Terminal window, create a web service project, replacing extensionname with the name of your extension that you defined earlier in step 1.

    ```bash
    dotnet new web -o extensionname
    ```

6. Next, change into the directory where your source lives (which will be the name of your extension, NOT the word extensionname) by typing the following and pressing Enter:

    ```bash
    cd extensionname
    ```

    You’ll now add a reference to the [GitHub Octokit](https://www.nuget.org/packages/Octokit) library which makes it easier to work with the GitHub API from C#.

7. Type the following and press Enter:

    ```bash
    dotnet add package Octokit --version 13.0.1
    ```

    There are additional Octokit libraries for JavaScript, Ruby, Go, and more. Visit [https://github.com/octokit](https://github.com/octokit) for more information.

8. Open Program.cs from the Explorer.

9. Modify the existing code in line 4 to say “Hello Copilot!” instead of the existing message. The new line should look like as follows:

    ```csharp
    app.MapGet("/", () => "Hello Copilot!");
    ```

10. Create a new record file called Message.cs with the following body:

    ```csharp
    public record Message
    {
        public required string Role { get; set; }
        public required string Content { get; set; }
    }
    ```

    Copilot will provide data containing the role (system, assistant, or user) and the message data.

11. Create a new record file called Request.cs with the following body:

    ```csharp
    public record Request
    {
        public bool Stream { get; set; }
        public List<Message> Messages { get; set; } = [];
    }
    ```

20. Back in Program.cs, add the following using directives to the top of the file:

    ```csharp
    using Microsoft.AspNetCore.Mvc;
    using Octokit;
    using System.Net.Http.Headers;
    ```

21. Still in Program.cs, add variables with their default values, making sure to replace extensionname with the real name of your extension. You will want this code to be above the line of code that shows app.Run();:

    ```csharp
    string yourGitHubAppName = "extensionname";
    string githubCopilotCompletionsUrl = "https://api.githubcopilot.com/chat/completions";
    ```

22. Still in Program.cs, add the following code to expose an agent endpoint for Copilot to access. You will want this code to be above the line of code that shows app.Run();:

    ```csharp
    app.MapPost("/agent", async ([FromHeader(Name = "X-GitHub-Token")] string githubToken, [FromBody] Request userRequest) =>
    {

    });
    ```

23. You can make sure you don’t have any typos or errors by going to the Terminal and running the following command:

    ```bash
    dotnet build
    ```

    Note, as you complete the rest of the steps, you will use this command to make sure you’ve not made any mistakes.

24. Identify the user using the GitHub API token provided in the request headers.

    ```csharp
    var octokitClient = new GitHubClient(new ProductHeaderValue(yourGitHubAppName))
    {
        Credentials = new Credentials(githubToken)
    };
    var user = await octokitClient.User.Current();
    ```

25. Now it’s time to insert special system messages. The first is to acknowledge the user using their GitHub login handle. The second is to have the extension ‘talk like Blackbeard the Pirate’. You’ll add them in the message list.

    ```csharp
    userRequest.Messages.Insert(0, new Message
    {
        Role = "system",
        Content = $"Start every response with the user's name, which is @{user.Login}"
    });
    userRequest.Messages.Insert(0, new Message
    {
        Role = "system",
        Content = "You are a helpful assistant that replies to user messages as if you were Blackbeard the Pirate."
    });
    ```

26. Use the HttpClient class to communicate back to Copilot. To keep the example simple, the code constructs one on every request. This is not optimal for production situations.

    ```csharp
    var httpClient = new HttpClient();
    httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", githubToken);
    userRequest.Stream = true;
    ```

27. Use Copilot's LLM to generate a response to the user's messages.

    ```csharp
    var copilotLLMResponse = await httpClient.PostAsJsonAsync(githubCopilotCompletionsUrl, userRequest);
    ```

28. The last bit of code needed is to stream the response straight back to the user.

    ```csharp
    var responseStream = await copilotLLMResponse.Content.ReadAsStreamAsync();
    return Results.Stream(responseStream, "application/json");
    ```

29. In order for a user to install your extension as a GitHub app, you need to provide a callback endpoint. Add the following code for the callback endpoint. You will want this code to be above the line of code that shows app.Run();:

    ```csharp
    app.MapGet("/callback", () => "You may close this tab and return to GitHub.com (where you should refresh the page and start a fresh chat). If you're using VS Code or Visual Studio, return there.");
    ```

30. Return to the Terminal window, and run the following command:

    ```bash
    dotnet build
    ```

    You should have 0 Warnings and 0 Errors. If you want, you can commit your changes, and optionally push them.

31. In the Terminal run the following command:

    ```bash
    dotnet run
    ```

    You will see a message pop up telling you that your application is running. You’ll see two choices. The first is Open in Browser. The second is Make Public.

32. Choose Open in Browser. A new tab will open and you should see your message “Hello Copilot!”

    This lets you quickly see that your service works. But, you need to do a bit more configuration before you can test the actual Copilot related code.

34. In the Terminal, press Ctrl+C to stop the app from running.

33. Let's create a dev tunnel to be able to access to our localhost from github. Execute the following command:

```
code tunnel
```

33. Open the like [https://github.com/login/device](https://github.com/login/device) with your browser and type the code from the command line. And authorize it.

33. Navigate to the provided url. This will open VSCode in the brower with the tunnel open.

33. It should propose to install the C# dev kit so please install it.

31. In the Terminal run the following command:

    ```bash
    dotnet run
    ```

32. Authorize the dev tunnel. You should see the message "Hello from copilot". 

33. Before you close the tab showing the message, copy the full URL and save it somewhere like a text file. Once you’ve saved it, close the tab and return to your Codespace.

34. In the Terminal, press Ctrl+C to stop the app from running.

35. Open a new browser tab and navigate to your GitHub account at [https://github.com](https://github.com/).

36. In the upper-right corner of any page on GitHub, click your profile photo.

37. You are configuring this for a personal account, so click Settings.

38. In the left sidebar, click Developer settings.

39. In the left sidebar, click GitHub Apps.

40. Click New GitHub App.

41. Under GitHub App name, enter a name for your app. This should be the name you chose in step 1 above.

    Reminder: The name cannot be longer than 34 characters.

    Your app's name will be shown in the user interface when your app takes an action. Uppercase letters will be converted to lowercase, with spaces replaced by -, and accents ignored. For example, My APp Näme would display as my-app-name.

    The name must be unique across GitHub. You cannot use the same name as an existing GitHub account, unless it is your own user or organization name.

42. Optionally, under Description, type a description of your app. Users and organizations will see this description when they install your app.

43. Under Homepage URL, enter a URL for your app.

    You can use:

    - Your app's website URL.
    - The URL of the person that owns the app (like their public GitHub page).
    - The URL of the repository where your app's code is stored, if it is a public repository.

44. Under Callback URL, which will be your server's hostname (aka forwarding endpoint) that you copied from your browser earlier. Because you’re using Codespaces, It will look something like this with callback at the end:

    ```
    https://funny-name-5abcde6zyxwvu7a-1111.app.github.dev/callback
    ```

    Naturally, this is only for development and testing. You’ll want a proper hosting environment for real use.

45. Under Webhook deselect Active.

    Webhooks let your app subscribe to events happening in a software system and automatically receive a delivery of data to your server whenever those events occur. Webhooks are used to receive data as it happens, as opposed to polling an API (calling an API intermittently) to see if data is available. Your extension will not use them which is why you’re unchecking this option.

46. Finally, click Create GitHub App.

    Now that you’ve created the app, you need to adjust its Permissions.

47. In the left sidebar, click Permissions & events.

48. Expand the Account permissions option.

    You only need one permission for your extension to be usable by a user via GitHub Copilot. If however, you want to access additional data on behalf of the user on GitHub, you’ll need to adjust the permissions accordingly. This should be part of your extension design and planning process.

49. In the dropdown menu, to the right of GitHub Copilot Chat item, select Read-only.

50. Scroll down and click Save changes.

51. In the left sidebar, click Copilot.

52. Under URL enter your server's hostname (aka forwarding endpoint) that you copied from your browser earlier. Because you’re using Codespaces, It will look something like. You need to add agent at the end like the following:

    ```
    https://funny-name-5abcde6zyxwvu7a-1111.app.github.dev/agent
    ```

53. In the Inference description, provide a string that you want the user to see if they access your extension from Visual Studio Code or Visual Studio 2022. For example: I talk like a pirate

    Note: In the future, this will be used for a machine-ready description, used to infer your App's capabilities.

54. Click Save.

55. In your GitHub App settings, in the left sidebar, click Install App.

56. Click the Install button next to your account (you account should be the only one visible at this time).

57. Leave this tab open.

58. Switch back to your Codespace.

59. In the Terminal and running the following command:

    ```bash
    dotnet run
    ```

    You will see a message pop up telling you that your application is running. You’ll see two choices again. The first is Open in Browser. The second is Make Public.

60. Choose Make Public.

    You can now test your extension from Copilot!

61. Follow along below.

    - If you have access to GitHub Copilot Enterprise, on any page on GitHub.com, click the GitHub Copilot icon on the page. The Copilot Chat panel is displayed.
    - If you don’t have a GitHub Copilot Enterprise license, or if you prefer to test from a client IDE, open the Copilot Chat panel in VS Code or Visual Studio 2022. This assumes you have either a GitHub Copilot Individual or GitHub Copilot Business license or trial.

62. Invoke your extension from the Copilot Chat panel by typing @EXTENSIONNAME, replacing any spaces in the extension name with -, then press Enter.

63. If this is your first time using the extension, you will be prompted to authenticate. Follow the steps on screen to authenticate your extension.

    Because you’re using a Codespace, you’ll get a warning from GitHub:

    You are about to access a development port served by someone’s codespace.

    This is your Codespace so it’s OK.

64. Click Continue.

65. Ask your extension a question in the chat window.

    For example, What is the software development lifecycle?

    You should get a very nice answer written in a “pirate voice”.

66. Ask your extension a non-programming question in the chat window. For example, Tell me a pirate dad joke.

    You will most likely be told “The response was filtered due to the prompt not being programming related. Please modify your prompt and retry.” This is expected as GitHub Copilot is focused on software development workflows and not general Q&A.

67. When you’re done testing, stop your running app.

68. Close your Codespace.

69. Go to [https://github.com/codespaces](https://github.com/codespaces) and stop your codespace.

    And there you have it. Your first GitHub Copilot Extension running as a GitHub App!

---

## Additional Links

- [Visit GitHub Resources](https://resources.github.com/)
- [Copilot plans](https://github.com/features/copilot/plans)
- [GitHub Octokit Documentation](https://github.com/octokit)
- [GitHub Copilot Extensions Guide](https://docs.github.com/en/copilot/building-copilot-extensions)

This workshop has been create from this blog post: [https://resources.github.com/learn/pathways/copilot/extensions/building-your-first-extension/](https://resources.github.com/learn/pathways/copilot/extensions/building-your-first-extension/)

Follow these steps to build and test your first GitHub Copilot Extension successfully!
