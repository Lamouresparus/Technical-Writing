
# Generate Images from Text in Android Using Replicate's Kotlin Client Library

With the current buzz in Artificial Intelligence (AI), more developers are excited to integrate AI into their applications to enhance productivity, solve problems and streamline workflows. 

Replicate simplifies this process by making it possible to run machine learning models with a cloud API. This way, you don't have to understand the intricacies of machine learning to use machine learning models in your applications. 

Earlier this year, I contributed to building this [Replicate Kotlin Client Library](https://github.com/enyason/replicate-kotlin), which provides a client class to interact with the Replicate API, offering methods for creating, retrieving, and cancelling predictions.

**In this article, you will use the Replicate Kotlin client class to generate images from text using the [Stable Diffusion text-to-image model](https://replicate.com/stability-ai/stable-diffusion/readme).**

**You will also:**
1. Get familiar with some Replicate terminologies
2. Create a Replicate Account and get your API token.
3. Create a Prediction and Use its output.

## Prerequisites
To successfully follow this article, you need the following:
* Basic Knowledge of Android development with Kotlin and Compose.
* Recent version of Android Studio. I will be using Android Studio Iguana in this article.
* A GitHub account. This will be used to create a Replicate account.

Let's get started!

> If you already have your Replicate API key and a project you want to use Replicate on, you can skip to **[Set Up the Project](#set-up-the-project)**.

## Some Replicate Terminologies
**Model:**  
&nbsp;&nbsp; In Replicate, a model is an already trained and published machine learning program that accepts an input, processes it using its underlying algorithm and returns an output. 

**Prediction:**  
&nbsp;&nbsp; Whenever you run a model, it creates a prediction. A prediction object will contain the output from the AI model, the input you provided, as well as other metadata. The image below shows the different properties of a Prediction. 

<img width="497" alt="Screenshot 2024-08-22 at 09 03 09" src="https://github.com/user-attachments/assets/35fc2d27-b68f-4e00-9e88-b51f18a0128c">


**Streaming:**  
&nbsp;&nbsp; Streaming makes it possible to receive real-time chunks of output while the model is still processing your input. This creates a seamless feel.

## Create a Replicate Account and Generate your API Key
1. Visit [replicate.com/signin](https://replicate.com/signin).
2. Click on **Sign in with GitHub**. If you are already signed in to Github, you will be asked to authorise Replicate to read your private email address. If you are not signed in on Github, you will be redirected to sign in to Github.
3. Authorise Replicate and proceed or skip the onboarding.
4. On the top-left corner of the screen, Click on your username. In the pop-up menu that appears, click on API tokens. 
<img width="300" alt="Screenshot 2024-08-22 at 15 43 25" src="https://github.com/user-attachments/assets/3c1a7f36-a0f2-436b-a03d-99f5db5a576b">

5. In The API tokens screen, Click on the copy icon that appears when you focus on the Token tile. You can also create a new token by entering a name in the Input token name field and clicking on the Create Token button.
<img width="500" alt="Screenshot 2024-08-22 at 15 51 25" src="https://github.com/user-attachments/assets/0890a083-49b5-4954-b7bd-2ab3ec4128d0">


## Create A Project In Android Studio.
1. Launch Android Studio.
2. Create a new Empty project.

## Set Up the Project 
1. **Enable Internet Access:**
In your Manifest.xml file, add internet permissions, to enable your app to access the internet.

```XML
<uses-permission android:name="android.permission.INTERNET" />
```
2. **Add Dependencies:**
In your module-level `build.gradle` file, add the following dependencies:
    - Coil for loading images from the internet:
      ``` Groovy
      implementation("io.coil-kt:coil-compose:2.7.0")
      ```
   - Replicate Kotlin Client Library:
     ``` Groovy
     implementation ("io.github.enyason:replicate-kotlin:0.1.0")
     ```
3. Sync Project with Gradle files.

## Setup the Layout
Since you will be generating an image from an inputted text, you will need:
- an input field, where the user will add their prompts,
- an image view to display the generated image, and
- a button to trigger image generation from the prompt.

You can also add views for loading and error states.

1. Create a file named `ImageGeneratorScreen`.
2. In a `Column`, add a `SubcomposeAsyncImage`, a `TextField`, and a `Button`. I added a `CircularProgressIndicator` to show the loading state.
Your `ImageGeneratorScreen` should look somewhat like this:

``` Kotlin
@Composable
fun ImageGeneratorScreen() {

    var imageUrl by remember { mutableStateOf("") }
    var prompt by remember { mutableStateOf("") }
    var isLoading by remember { mutableStateOf(false) }

    Column(
        modifier = Modifier
            .padding(24.dp)
            .fillMaxWidth(),
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        if (isLoading) {
            CircularProgressIndicator(modifier = Modifier.align(Alignment.CenterHorizontally))
        } else {
            Spacer(modifier = Modifier.height(24.dp))

            if (imageUrl.isNotBlank()) {
                SubcomposeAsyncImage(
                    model = imageUrl,
                    modifier = Modifier
                        .fillMaxWidth()
                        .wrapContentHeight()
                        .border(2.dp, Color.Gray, RoundedCornerShape(4.dp)),
                    loading = {
                        Text(
                            text = "Image is loading...",
                            textAlign = TextAlign.Center,
                            modifier = Modifier.padding(vertical = 16.dp, horizontal = 8.dp)
                        )
                    },
                    error = {
                        Text(
                            text = "Error Loading Image ${it.result.throwable.localizedMessage}",
                            textAlign = TextAlign.Center,
                            modifier = Modifier.padding(vertical = 16.dp, horizontal = 8.dp)
                        )
                    },
                    contentDescription = ("Generated Image")
                )
            } else {
                Text(
                    text = "Your Image Will Appear Here",
                    textAlign = TextAlign.Center,
                    modifier = Modifier
                        .border(2.dp, Color.Black, RoundedCornerShape(8.dp))
                        .padding(vertical = 48.dp)
                        .fillMaxWidth()
                )
            }


            Spacer(modifier = Modifier.height(24.dp))

            TextField(
                value = prompt,
                onValueChange = { prompt = it },
                placeholder = {
                    Text("Enter your prompt")
                },
                modifier = Modifier.fillMaxWidth()
            )
            Spacer(modifier = Modifier.height(32.dp))

            Button(onClick = {}) {
                Text("Submit")
            }
        }
    }
}
```
2. In your `MainActivity`, call `ImageGeneratorScreen()`.
3. Run the app, and your UI should look similar to this:
<img width="170" alt="Screenshot 2024-08-22 at 16 22 52" src="https://github.com/user-attachments/assets/6840ea89-81e4-4e02-80d5-b2980c2462d5">


## Use The Replicate Client

### Step 1: Create the Replicate client Object.

You can do this in two ways:

1. **Using just your API token:**
``` Kotlin
 val replicateClient = Replicate.client("API Token")
```
 2. **Using a Configuration object:**

``` Kotlin
val config = ReplicateConfig(apiToken = "API Token", pollingDelayInMillis = 3000)
val replicateClient = Replicate.client(config)
```

Using the configuration object gives you more control over how the library behaves. You can enable or disable logging, specify the kind of information you want on the logs, and also set the time delay in milliseconds for polling the server. 

> [!WARNING]
> Do not check your API token into version control to protect it from malicious use. One way to do this is to store it in a local `secrets.properties` file and use the [Secrets Gradle plugin](https://developers.google.com/maps/documentation/android-sdk/secrets-gradle-plugin) to access the API key.

**Why do We need A polling delay?**

*This is because while most models run quickly, some models like the generative model you will be using in this article may take a longer time to run, which creates the need to poll the API, to check the status of the prediction. I found this [Quora answer](https://qr.ae/p2vQnK) on Polling quite interesting.*


### Step 2: Create a Prediction.
To create a `Prediction`, you need a `Predictable` object.
This object will specify:
- the model you want to run: by its version and model IDs,
- the input you want to be processed, and
- other arguments to tweak the output of the model.

#### Set Up the ImageGenerator Class

1. Create an `ImageGenerator` Class that implements `Predictable`. The information from this class will be used to build the request body that will be sent to the replicate API.

Your class should look like this:
``` Kotlin
data class ImageGenerator(
    override val versionId: String,
    override val input: Map<String, Any>,
    override val modelId: String? = null,
) : Predictable

```


The `ImageGenerator` has 3 properties:
- `versionId`: The version ID of the model,
- `input`: A map that will contain the prompt and other arguments.
- `modelId`: The ID of the model (optional).

As stated earlier, you will be using the Stable-Diffusion model to generate an image from text. 

You can find the latest version ID [here](https://replicate.com/stability-ai/stable-diffusion/versions). The latest version ID at the time of writing is: 

`ac732df83cea7fff18b8472768c88ad041fa750ff7682a21affe81863cbe77e4`

Before creating the input map, you can check Stable-Diffusion's input schema to know what properties are required and the values they should contain.

<img width="500" alt="Screenshot 2024-08-22 at 10 16 12" src="https://github.com/user-attachments/assets/5462f1f1-1203-4737-88fc-3eac928ac294">

You can find the complete input schema [here](https://replicate.com/stability-ai/stable-diffusion/api/schema#input-schema).

For this article, I will only specify the:
- `prompt`,
- `width`, and
- `height` as inputs.


You can choose to add more as you please.

2. In the `ImageGenerator` class, pass the model's version ID as a default value for the `versionId` property. 

Since the `height` and `width` will have a static value, you can create a function, that will modify just the `prompt` in the map. 

3. Create a function inside the `companion object` of the `ImageGenerator` class that takes a prompt as an argument and returns a Map containing the input parameters. The `width` and `height` should have a static value of `768`, and the prompt should be passed as an argument.

   
Your `ImageGenerator` class should look like this:

``` Kotlin

data class ImageGenerator(
    override val versionId: String = "ac732df83cea7fff18b8472768c88ad041fa750ff7682a21affe81863cbe77e4",
    override val input: Map<String, Any>,
    override val modelId: String? = null,
) : Predictable {

    companion object {
        fun createInputMap(prompt: String): Map<String, Any> {
            return mapOf(
                "width" to 768,
                "height" to 768,
                "prompt" to prompt
            )
        }
    }
}

```


#### Create a Prediction in the Button's onClick Function

1. In the `ImageGeneratorScreen`, within the `onClick` lambda function of the Button, launch a coroutine scope when the prompt is not blank.
2. Within the coroutine scope, call the `createInputMap()` function, pass in the prompt, and store its value in a variable.
3. Next, call `replicateClient.createPrediction`.

Notice that `createPrediction` expects a generic type parameter. This parameter represents the data type of the Prediction output.

> The generic type is necessary because of type erasure that occurs at compile time. By using reified generics, the type information is preserved, allowing the function to operate on the specific type you provide.

Based on the output schema snippet below, the expected output is a `List<String>` containing URIs of the generated images.

<img width="350" alt="Screenshot 2024-08-22 at 13 11 48" src="https://github.com/user-attachments/assets/35faff30-35ed-4a24-a2bc-5c5db342b35f">

4. Now pass in `List<String>` as the generic type parameter for the `createPrediction` function and use the generated `inputMap` as the input property.

The `createPrediction()` function returns a `Task` object representing the created prediction. 

5. Call `await()` on this object to wait for the task to complete and retrieve the Prediction.
6. If the prediction output is not empty or null, load the first Uri in the list.

The Button's `onClick` lambda function should look somewhat like this:

``` Kotlin
{
    if (prompt.isNotBlank()) {
        isLoading = true
        CoroutineScope(Dispatchers.IO).launch {
            val inputMap = ImageGenerator.createInputMap(prompt)
            val predictable = replicateClient.createPrediction<List<String>>(
                ImageGenerator(input = inputMap)
            )
            val result = predictable.await()
            val output = result?.output ?: emptyList()
            if (output.isNotEmpty()) {
                isLoading = false
                imageUrl = output.first()
            }
        }
    }
}

```
## Running the App

Run your app and type in a prompt.

Voila! Everything should work as expected.

Here is a GIF of the app's interaction: [Click Here](https://github.com/user-attachments/assets/75991c8c-e080-475d-aa49-08e33f358474)

You can find the GitHub gist [here](https://gist.github.com/Lamouresparus/5e56f57adbab46194edc8a2aa99cb9cc)

Thank you















