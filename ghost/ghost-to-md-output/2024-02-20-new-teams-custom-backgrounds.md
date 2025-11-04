---
title: New Teams custom backgrounds
slug: new-teams-custom-backgrounds
date_published: 2024-02-20T10:51:55.000Z
date_updated: 2024-02-20T10:51:55.000Z
tags: macOS, Shell script, Teams
excerpt: Custom backgrounds in Teams offer an excellent opportunity to enhance company branding or promote internal initiatives.
---

Custom backgrounds in Teams offer an excellent opportunity to enhance company branding or promote internal initiatives. However, the recent update to Teams on macOS has altered the deployment process, particularly concerning the location of these backgrounds. In response to requests for background deployment, I've developed a script that can be executed from Intune, Munki, or any platform of your choice. Importantly, this script can be run as either root or the current user, offering flexibility in its application.

In addition the script has some features that makes management easier for you.

- The file can be either an image file or zip file, allowing you to easily deploy multiple backgrounds in one go.
- The thumb image is automatically resized and cropped with correct aspect ratio based on the background image.
- If the image format is not PNG, it is automatically converted to a PNG.

Let's have a look at the script step by step which can be found [here](https://github.com/almenscorner/macOS/blob/main/Scripts/Teams/teamsbackground.sh).

First, some required parameters are set which should not be changed. These gets the current user and the home folder, creates a temp directory and sets the path to the backgrounds folder:

    ### Do not modify ###
    CURRENT_USER=$( echo "show State:/Users/ConsoleUser" | scutil | awk '/Name :/ { print $3 }' )
    USER_HOME=$(dscl . -read /users/${CURRENT_USER} NFSHomeDirectory | cut -d " " -f 2)
    TMPDIR=$(mktemp -d)
    BACKGROUND_FOLDER="$USER_HOME/Library/Containers/com.microsoft.teams2/Data/Library/Application Support/Microsoft/MSTeams/Backgrounds/Uploads"
    

To run the script, you only have to set one configuration which is the URL to where the image or zip should be downloaded:

    ### Required settings ###
    BACKGROUND_URL="" # Replace with the URL of the image or zip you want to download
    

There is also some option configurations such as setting a variable to do a run time check if the script has been run before and should not run again:

    ### Optional settings ###
    IMAGE_FORMATS=("png" "jpg" "jpeg") # Add or remove image formats as needed
    RUN_CHECK=false # Set to true to prevent the script from running more than once
    RUN_CHECK_FILENAME=".teamsbackground" # Name of the file to check if the script has been run before
    RUN_CHECK_PATH="$USER_HOME/Library/Application Support/$RUN_CHECK_FILENAME" # Path to the file to check if the script has been run before
    

The `process_image` function is responsible for handling the extraction of the zip file if it is one and then process each image file found:

    process_image() {
        local f="$1"
        file_extension="${f##*.}"
        for format in "${IMAGE_FORMATS[@]}"; do
            if [[ "$file_extension" == "$format" ]]; then
                IMAGE_GUID=$(uuidgen)
                IMAGE_PATH="$BACKGROUND_FOLDER/$IMAGE_GUID.png"
                IMAGE_THUMB_PATH="$BACKGROUND_FOLDER/${IMAGE_GUID}_thumb.png"
                # If not a png, sips convert it to png in current folder  
                if [[ $f != *.png ]]; then
                    sips -s format png "$f" -o "$f.png" > /dev/null
                fi
    
                # Rename the file to the GUID in current folder
                mv "$f" "$IMAGE_GUID"
                # Copy the image to the background folder
                cp "$IMAGE_GUID" "$IMAGE_PATH"
                # Copy the thumb file to the background folder
                cp "$IMAGE_GUID" "$IMAGE_THUMB_PATH"
    
                # Resize the image to have a height of 186, maintaining the aspect ratio
                sips -Z 186 "$IMAGE_THUMB_PATH" -o "$IMAGE_THUMB_PATH" > /dev/null 2>&1
                # Get the current width of the image
                width=$(sips -g pixelWidth "$IMAGE_THUMB_PATH" | awk '/pixelWidth:/{print $2}')
                # Calculate the amount to crop from the sides
                crop=$((($width - 238) / 2))
                # Crop the image to have a width of 238, centered
                sips -z 186 238 "$IMAGE_THUMB_PATH" -o "$IMAGE_THUMB_PATH" > /dev/null 2>&1
    
                log_message "Background image downloaded and set, image GUID: $IMAGE_GUID"
            fi
        done
    }
    

Running the script, the image or zip will be downloaded and backgrounds and thumbs will be created:

    [Tue Feb 20 10:59:16 CET 2024] - Background image downloaded and set, image GUID: F4D42A93-7551-46E3-B0F7-DB98A12A493B
    [Tue Feb 20 10:59:20 CET 2024] - Background image downloaded and set, image GUID: BC0390C6-E800-4786-937D-2D5709DD0FD4
    [Tue Feb 20 10:59:23 CET 2024] - Background image downloaded and set, image GUID: 3D2DBDE1-3450-4939-AB0B-33CD45929121
    

If `RUN_CHECK` has been set to `true` and the script has run once before, it will not run again:

    [Tue Feb 20 11:43:17 CET 2024] - Script has already been run, exiting
    

Quick post but hopefully this makes it easy for you to deploy custom Teams backgrounds to users using the new Teams.
