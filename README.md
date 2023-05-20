# F-Droid-Helper
Simple instructions for adding an app to F-Droid. For more instructions visit [the official Gitlab repo](https://gitlab.com/fdroid/fdroiddata/blob/master/CONTRIBUTING.md). Goal of the steps below is to create a merge request on their offical repo so they can add your app.

## Steps
### 1. Prepare your app (example app is hosted in Github)
* Add fastlane structure with descriptions, icon, title, screenshots etc for the F-Droid store:
  * In the root directory of your app create the following folder structure in `fastlane/metadata/android/`([source](https://f-droid.org/en/docs/All_About_Descriptions_Graphics_and_Screenshots/)). For an example see [here](https://github.com/amnesica/KryptEY/tree/master/fastlane/metadata/android/en-US). Not all entries are mandatory (in my case having `short_description.txt`, `full_description.txt`, `title.txt`, changelog file, `icon.png` and screenshots in the resp. directories was enough)
  ```
  ├── en-US                       (en-US is the F-Droid fallback language)
  │   ├── short_description.txt   (short description, max 80 chars, mandatory)
  │   ├── full_description.txt    (full app description, mandatory)
  │   ├── title.txt               (app name)
  │   ├── video.txt               (URL to a video introducing the app)
  │   ├── images
  │   │   ├── icon.png            (app icon, mandatory if your app doesn't include any png icon)
  │   │   ├── featureGraphic.png  (promo banner, shown on top of the app desc in F-Droid client; landscape)
  │   │   ├── tvBanner.png        ("icon" for TV devices, currently not used)
  │   │   ├── phoneScreenshots
  │   │   │   ├── 1.png
  │   │   │   ├── 2.png
  │   │   │   ...
  │   │   ├── sevenInchScreenshots/
  │   │   ├── tenInchScreenshots/ (you may add different screenshots for different screen sizes)
  │   │   ├── tvScreenshots/
  │   │   └── wearScreenshots/
  │   └── changelogs
  │       ├── 100000.txt          (must correspond to versionCode, literally, no padding)
  │       ├── 100100.txt          (if the version code was set to 100100)
  │       └── 100101.txt          (maximum size: 500 characters)
  └── ru                          (other locale codes)
      ...                         (localized metadata is always preferred by the client)
      └── changelogs
          └── 100100.txt
  ```
* Build signed apk in Android studio (optional: if you want reproducible builds on F-Droid make sure to build the apk with Java 11 OpenJDK (Go to your Android studio settings and select `java-11-openjdk` for the Gradle JDK under `Build, Execution, Deployment -> Build Tools -> Gradle`))
* Create a tagged release on Github and add signed apk file 
  * Create the tag with `v1.0.0` for example. The prefix `v` is important in my case (as we will see later)
* (optional) Get checksum of signed apk file (needed later for reproducible builds on F-Droid!)
  * Make sure your apk was build with `java-11-openjdk`! (see step above)
  * Go to your local android installation folder (`/home/YOUR-USERNAME/Android/Sdk/build-tools`) and select the right build-tools version (in my case its 33.0.0). Go into the `/lib` directory. You should see the `apksigner.jar`
  * If you want you can create an `alias` to the `apksigner.jar` which is needed for creating the checksum: I'm using `zsh` so I'm going into my `.zshrc` and pasting `alias apksigner="java -jar /home/YOUR-USERNAME/Android/Sdk/build-tools/33.0.0/lib/apksigner.jar"`. Save and `source` the `.zshrc` file. 
  * Go to your signed apk and use `apksigner verify --print-certs YOUR-SIGNED-APK-NAME.apk | grep SHA-256`
  * Save the output for later! (in my case its `Signer #1 certificate SHA-256 digest: 8e3b3bfb8308fa4cd14b9a32deb31e0b15106d25eb6258b6e25c963bebf8b3ee`)

### 2. Add app metadata to F-Droid repo (without fdroidserver)
1. Create a Gitlab account
2. Visit and fork the [fdroiddata repository](https://gitlab.com/fdroid/fdroiddata)
3. Visit your fork
4. Create a new branch
5. Naming it like the app name or, much better, the app id makes it easier to keep track of your contributions.
6. Visit the `metadata` directory of your fork.
7. Add a new file by clicking on the plus sign and choosing *"New file"*.
8. Set the file name in the following schema: `<application_id>.yml`. So an example would be *"com.app.example.yml"*.
9. Write down the metadata. The [Build Metadata Reference](https://f-droid.org/en/docs/Build_Metadata_Reference) as well as the [templates](https://gitlab.com/fdroid/fdroiddata/-/blob/master/templates/README.md) will help you. For simplicity I will use my own example metadata file for [KryptEY](https://github.com/amnesica/KryptEY) and add some explanations (please remove them if you're using this as a template!!):
```
Categories:
  - Security                                                                                              # Choose one or multiple existing categories (`Connectivity`, `Development`,  `Games`, `Graphics`, `Internet`, `Money`, `Multimedia`, `Navigation`, `Phone & SMS`, `Reading`, `Science & Education`, `Security`, `Sports & Health`, `System`, `Theming`, `Time`, `Writing`). Create another `- ExampleCategory` when you wish to add another category
License: GPL-3.0-only                                                                                     # Add your license here!
AuthorName: amnesica                                                                                      # Your name 
SourceCode: https://github.com/amnesica/KryptEY                                                           # Link to your repo
IssueTracker: https://github.com/amnesica/KryptEY/issues                                                  # Link to the issues page of your repo
Changelog: https://github.com/amnesica/KryptEY/releases                                                   # Link to the release page of your repo

AutoName: KryptEY                                                                                         # App name

RepoType: git                                                                                             # What kind of git it is 
Repo: https://github.com/amnesica/KryptEY.git                                                             # Link to your repo with `.git` ending
Binaries: https://github.com/amnesica/KryptEY/releases/download/v%v/com.amnesica.kryptey_v%v-release.apk  # (optional if you want reproducible builds!) Generalized link to your signed apk (when you use the tag `vX.X.X` you can just reuse this link here and change app name and so forth)

Builds:
  - versionName: 0.1.5                                                                                    # Version name without `v` (see your build.gradle)
    versionCode: 24                                                                                       # Version code without (see your build.gradle)
    commit: v0.1.5                                                                                        # Latest tag of version (specifies the tag, commit or revision number from which to build it in the source repository)
    subdir: app                                                                                           # Latest tag of version
    gradle:
      - yes

AllowedAPKSigningKeys: 8e3b3bfb8308fa4cd14b9a32deb31e0b15106d25eb6258b6e25c963bebf8b3ee                   # (optional if you want reproducible builds!) Checksum of the apksigner output (see step above)

AutoUpdateMode: Version                                                                                   # Method used for auto-generating new builds when new releases are available
UpdateCheckMode: Tags v.*                                                                                 # Method using for determining when new releases are available
CurrentVersion: 0.1.5                                                                                     # Name of the version that is the recommended release
CurrentVersionCode: 24                                                                                    # Version code corresponding to the CurrentVersion field
```
11. Choose a smart commit message and commit your changes.
12. Go to the `CI/CD` menu in the GitLab project of your fork.
13. Check if the pipeline for your commit(s) succeed. If not, take a look into the logs and try to fix the error by editing the metadata file again.
  * If there are errors look at the pipeline and try to fix them
15. If everything went fine, you can create a new [merge request](https://gitlab.com/fdroid/fdroiddata/-/merge_requests) at the `fdroiddata` repository.
16. Now wait for the packagers to pick up your Merge Request. Please keep track if they asked any questions and reply to them as soon as possible.
