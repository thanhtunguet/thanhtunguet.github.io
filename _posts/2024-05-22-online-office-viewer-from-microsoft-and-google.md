---
title: "Opening Document Files Using Online Viewers from Google and Microsoft"
date: 2024-05-22 00:01:00 +0700
categories: [Programming]
tags: [Programming, "Office Viewer"]
pin: false
---

As a developer, enabling seamless access to document files like Office documents and PDFs is a common requirement. Online viewers from Google and Microsoft provide an efficient way to open and display these files without requiring users to have the specific software installed. In this article, we'll explore how to use Google’s Gview and Microsoft’s Office Viewer to open document files in your applications.

## Using Google Gview

Google's Gview allows you to embed and view a variety of document types within your web application. This is particularly useful for PDFs and Office files (Word, Excel, PowerPoint).

### Steps to Use Google Gview

1. **Upload Your Document to a Public URL:**
   Ensure your document is accessible via a direct link. The document must be publicly accessible for the viewer to fetch and display it.

2. **Generate the Gview URL:**
   Use the following URL format to create the Gview link:
   ```
   https://docs.google.com/gview?url=YOUR_DOCUMENT_URL&embedded=true
   ```
   Replace `YOUR_DOCUMENT_URL` with the direct link to your document.

3. **Embed the Viewer in Your Webpage:**
   Use an `<iframe>` to embed the viewer in your webpage.
   ```html
   <iframe src="https://docs.google.com/gview?url=https://example.com/yourfile.pdf&embedded=true" style="width:600px; height:500px;" frameborder="0"></iframe>
   ```

### Example

Let's say you have a PDF file located at `https://example.com/sample.pdf`. The embedded viewer would look like this:
```html
<iframe src="https://docs.google.com/gview?url=https://example.com/sample.pdf&embedded=true" style="width:600px; height:500px;" frameborder="0"></iframe>
```

## Using Microsoft Office Viewer

Microsoft offers a similar service that allows you to view Office documents online. This service is particularly useful for Word, Excel, and PowerPoint files.

### Steps to Use Microsoft Office Viewer

1. **Upload Your Document to a Public URL:**
   Like Google Gview, ensure your document is accessible via a direct link and is publicly accessible.

2. **Generate the Office Viewer URL:**
   Use the following URL format to create the Office Viewer link:
   ```
   https://view.officeapps.live.com/op/embed.aspx?src=YOUR_DOCUMENT_URL
   ```
   Replace `YOUR_DOCUMENT_URL` with the direct link to your document.

3. **Embed the Viewer in Your Webpage:**
   Use an `<iframe>` to embed the viewer in your webpage.
   ```html
   <iframe src="https://view.officeapps.live.com/op/embed.aspx?src=https://example.com/yourfile.docx" width="600px" height="500px" frameborder="0"></iframe>
   ```

### Example

Let's say you have a Word document located at `https://example.com/sample.docx`. The embedded viewer would look like this:
```html
<iframe src="https://view.officeapps.live.com/op/embed.aspx?src=https://example.com/sample.docx" width="600px" height="500px" frameborder="0"></iframe>
```

## Practical Considerations

- **Public Access:** Ensure your documents are hosted on a server that allows public access. If your documents are behind authentication, these viewers won’t be able to fetch and display the files.
- **Document Types:** Google Gview supports a wider range of document types including PDFs, whereas the Microsoft Viewer is specialized for Office documents.
- **Styling and Dimensions:** Adjust the `width` and `height` attributes of the `<iframe>` to fit your web page design. 

## Conclusion

Embedding documents using Google’s Gview and Microsoft’s Office Viewer provides a simple and effective way to display documents directly on your website or web application. By following the steps outlined above, you can easily integrate these viewers and enhance the user experience by allowing seamless document viewing capabilities.

Feel free to experiment with both viewers and choose the one that best fits your needs and the types of documents you commonly work with. Happy coding!