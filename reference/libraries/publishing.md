# AppVeyor/GitHub/nuget.org pipeline configuration

The following guide covers the necessary steps to get a working AppVeyor project which monitors a Github C#/VL repository and whenever a push is made to it:

1. Clones the repository
2. Builds the source code
3. Generates a nuget package
4. Publishes the package to nuget.org
5. Notifies you about results via e-mail

The guide assumes you already have created an account for GitHub, AppVeyor and nuget.org and that you have access to an existing C#/VL GitHub repository.

## References

The explained configuration is currently being used for libraries such as:

* [VL.Devices.Kinect2](https://github.com/vvvv/VL.Devices.Kinect2)
* [VL.Devices.Nuitrack](https://github.com/vvvv/VL.Devices.NuiTrack)
* [VL.Devices.RealSense](https://github.com/vvvv/VL.Devices.RealSense)
* [VL.Devices.uEye](https://github.com/vvvv/VL.Devices.uEye)
* [VL.IO.NetMQ](https://github.com/vvvv/VL.IO.NetMQ)
* [VL.OpenCV](https://github.com/vvvv/VL.OpenCV)

Feel free to use them as starting points for your own library.

## nuget.org

The following steps will guide you through the nuget.org configuration, before proceeding please make sure you have a working account and are logged-in in nuget.org.

### Getting an API Key

1. Click on your user name at the top right
2. Click on `API Keys` in the menu that pops up
3. Click on `+ Create`
4. Under `Key Name` type the name of your repository or project, this is going to be the official package name for the world when they type `nuget install <YourPackageName>`
5. Under `Package owner` make sure you select the appropriate option depending on your case, if the package should belong to an organization you are part of rather than to you, choose said organization now
6. Under `Glob Pattern` type: *
7. Click `Create`

NOTE: At this point you should see your newly created package listed with a yellow warning message reminding you to copy your key, ***this is a crucial step as this is the only time you will be able to copy this value***. Click on `Copy` below your package's description and save the key in a safe place, it will be needed in step [Encrypting your nuget.org API Key](#Encrypting your nuget.org API Key).

## AppVeyor

The following steps will guide you through the AppVeyor configuration, before proceeding please make sure you have a working account and are logged-in in appveyor.com.

### Creating a new project

1. Click on `Projects` in the top menu
2. Click on `New Project`
3. Select repository, in this case we are using GitHub
4. Authorize as `OAuth App` unless you know what you are doing
5. Select `Public` or `Private and Public` depending on your preference
6. Click on `Authorize GitHub`
7. On the pop-up displayed click on `Authorize appveyor`

Once you have authorized AppVeyor you should be able to see a list with your available GitHub repositories, select the desired repository and click on `+ ADD`. After a few seconds you should see your new project listed in the `Projects` page.

### Encrypting your nuget.org API Key

1. Click on `Account` in the top menu
2. Select `Encrypt YAML` from the left menu
3. Paste your nuget.org key obtained in step [Getting an API Key](#Getting an API Key)
4. Copy the encrypted string in a safe place, it will be needed later in step [YAML file (appveyor.yml)](#YAML file (appveyor.yml))

## Configuration files

### File locations

In order to have full control over configuration, versioning numbers and other specific settings for your package, you need to avoid having your nuspec file in the project's root directory, otherwise AppVeyor will try to configure the package based on the nuspec file.

This step is not mandatory since you could find a different solution to get your configuration to work, however it has proved to be both effective and easy to grasp.

For everything to work correctly make sure you place your YAML file in the root of your project, and move your nuspec file into a subdirectory inside the root called `appveyor`.

Your project directory structure should in the end look somewhat like this:

    /
    /appveyor/<YourPackageName>.nuspec
    /appveyor.yml
    ...

For reference nuspec and YAML files you can check out the working copies for any of the GitHub repositories mentioned in [References](#references).

### YAML file (appveyor.yml)

#### nuget.org encrypted key

In order for AppVeyor to be able to push to your nuget.org's package feed, you will need to provide it with the encrypted nuget API Key obtained in step [Encrypting your nuget.org API Key](#Encrypting your nuget.org API Key).

To do so at the beginning of your YAML file make sure you have a section with the configuration below:

    environment:
	     nuget_api_key:
		       secure:<YourAPIKey>

Adjust your API key accordingly.

#### Release vs Pre-release packages

A nuget package can come in two versions, a release version or a pre-release version.

A pre-release package implies that the package is currently under development and things can change drastically from one version to the next. Functionality can also be expected to break or become unstable from time to time.

A release package implies that the package has been properly tested and polished for production. No major breaking changes are expected to happen and stability within the package should be reliable.

If you want to publish a pre-release version of your package you need to instruct nuget.org that this is indeed a pre-release version. To do this you must add the `-alpha` suffix at the end of your package's name.

This should be done in two sections within the YAML file:

##### assembly_info section:

Make sure to add the `-alpha` suffix at the end of your `assembly_version` attribute like so:

    assembly_version: "0.1.{build}-alpha"

##### after_build section:

Make sure to add the `-alpha` suffix at the end of your `nuget pack` and your `nuget push` command lines like so:

    after_build:
    - cd..
    - nuget pack appveyor\<YourProjectName>.nuspec -Version %APPVEYOR_BUILD_VERSION%-alpha`
    - nuget setApiKey %nuget_api_key%`
    - nuget push C:\projects\<YourProjectName>\<YourProjectName>.%APPVEYOR_BUILD_VERSION%-alpha.nupkg -Source https://api.nuget.org/v3/index.json`

If you do not want a pre-release package but rather a release package just remove any `-alpha` suffixes from your YAML file.

#### E-mail notifications

You can configure AppVeyor to send you an e-mail notification after it has finished processing your project, regardless of the outcome.

To do this add the following section at the end of your YAML file and change the values accordingly:

    notifications:
    - provider: Email to: - <yourEmail@server.org> subject: '<YourProjectName> Build {{status}} message: "{{message}}, {{commitId}}, ..." on_build_status_changed: true

### nuspec file

#### nuget dependencies

In the nuspec file, make sure you list the nugets needed by your library/project under the `dependencies` section.

#### Assets, binaries, help files, etc.

In the nuspec file, make sure you list any assets, dlls, help patches, etc. under the `files` section.

## Testing and Deployment

Once all previous steps have been successfully completed, you should be able to just push to the master branch of your GitHub repository and AppVeyor should start a build in a matter of a minute or two, you can then see the build process console in AppVeyor itself (click on your project in the projects list).

If nothing fails during the nuget restore/build/nuget package/push to nuget.org process you should see a green Build completed message at the end.

After a build succeeds, the newly generated nuget is pushed to nuget.org and you can follow its status there.

It usually it takes 10 to 15 minutes for a new nuget to get validated, indexed and listed.  Until it is both indexed an listed it will not be usable for anyone through `nuget.exe`.

## Troubleshooting

If anything fails during the process you should be able to see details on the specific error in AppVeyor's console window.
