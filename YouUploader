import os
import google_auth_oauthlib.flow
import googleapiclient.discovery
import googleapiclient.errors
import torch
from comfy.nodes import Node

class YouTubeUploaderNode(Node):
    """
    A custom node for comfyUI that uploads an input video to YouTube using the YouTube API.
    """
    def __init__(self, video_title, video_description, video_tags, client_id, client_secret):
        super().__init__()
        self.video_title = video_title
        self.video_description = video_description
        self.video_tags = video_tags
        self.client_id = client_id
        self.client_secret = client_secret

        # Set up YouTube API client
        scopes = ["https://www.googleapis.com/auth/youtube.upload"]
        client_config = {
            "installed": {
                "client_id": self.client_id,
                "client_secret": self.client_secret,
                "redirect_uris": ["urn:ietf:wg:oauth:2.0:oob", "http://localhost"],
                "auth_uri": "https://accounts.google.com/o/oauth2/auth",
                "token_uri": "https://oauth2.googleapis.com/token"
            }
        }
        flow = google_auth_oauthlib.flow.InstalledAppFlow.from_client_config(client_config, scopes)
        credentials = flow.run_console()
        self.youtube = googleapiclient.discovery.build(
            "youtube", "v3", credentials=credentials)

    def forward(self, input_tensor):
        # Save the input tensor as a video file
        video_file = "input_video.mp4"
        torch.save(input_tensor, video_file)

        # Upload the video to YouTube
        request = self.youtube.videos().insert(
            part="snippet,status",
            body={
                "snippet": {
                    "title": self.video_title,
                    "description": self.video_description,
                    "tags": self.video_tags,
                    "categoryId": "22"  # Video game category
                },
                "status": {
                    "privacyStatus": "private"
                }
            },
            media_body=video_file
        )
        response = request.execute()

        print(f"Video uploaded successfully: https://www.youtube.com/watch?v={response['id']}")
        os.remove(video_file)
        return input_tensor
