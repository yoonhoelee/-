# Handling Live Stream File URLs in a Spring Service

In this post, we'll look at how to modify a file service in a Spring application to handle live stream file URLs. When dealing with live streams, we often need to return a different URL format while keeping the rest of the file logic intact. Here's how you can achieve this.

---

## The Scenario

We have a file service that:
- Retrieves file information from a database.
- Generates file URLs for serving content.
- Needs special handling for live stream files to return a specific `liveStreamUrl`.

---

## The Updated `getFileUrl` Method

Here's the updated method:

```java
public String getFileUrl(Long fileId) {
    if (fileId == null) {
        throw new CodeException(DatabaseErrorCodes.FILE_NOT_EXIST);
    }

    FileInfo fileInfo = getFileById(fileId);

    // Check if the source location is LIVE_STREAM
    if ("LIVE_STREAM".equals(fileInfo.getSourceLocation())) {
        StringBuilder url = new StringBuilder(S3Constants.LIVE_STREAM_PATH)
            .append(fileInfo.getPath())
            .append("/")
            .append(fileInfo.getFileName())
            .append(getFileExtensionForLiveStream(fileInfo.getMimeType()));

        // Use the live stream CloudFront URL converter
        return CloudFrontUrlConverter.convertLiveUrl(url.toString());
    }

    return getFileUrl(fileInfo);
}
```
# Explanation of the Code
	1.	Null Check for fileId:
	•	Ensures that the provided file ID is valid and throws an appropriate exception if it isn’t.
	2.	Handle Live Stream Files:
	•	If the file’s source location is LIVE_STREAM, construct the URL using the live stream base path and file metadata.
	3.	Use a Separate URL Converter:
	•	For live stream files, CloudFrontUrlConverter.convertLiveUrl is called to generate the correct CloudFront URL.
	4.	Fallback for Non-LiveStream Files:
	•	For all other files, the method defaults to the existing logic, ensuring consistency.

# Example Output

Live Stream File
	•	Input:
	•	fileId: 123
	•	sourceLocation: LIVE_STREAM
	•	path: 2025/01/13
	•	fileName: sample
	•	mimeType: video/mp4
	•	Output:
```
https://live-stream.cloudfront.net/2025/01/13/sample.mp4
```
## Regular File
	•	Non-live stream files will continue to use the original URL generation logic.
# Benefits of This Approach
	•	Separation of Concerns:
	•	Logic for live stream files is isolated without impacting other file types.
	•	Extensibility:
	•	Easily adapt for other special file handling requirements.
	•	Reusability:
	•	Leveraging utility classes (CloudFrontUrlConverter) keeps the service clean and focused.
