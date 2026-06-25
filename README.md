# Teldrive
[![Ask DeepWiki](https://deepwiki.com/badge.svg)](https://deepwiki.com/tgdrive/teldrive)

Teldrive is a powerful utility that enables you to organise your telegram files and much more.

## Advantages Over Alternative Solutions

- **Exceptional Speed:** Teldrive stands out among similar tools, thanks to its implementation in Go, a language known for its efficiency. Its performance surpasses alternatives written in Python and other languages, with the exception of Rust.

- **Enhanced Management Capabilities:** Teldrive not only excels in speed but also offers an intuitive user interface for efficient file interaction which other tool lacks. Its compatibility with Rclone further enhances file management.

> [!IMPORTANT]
> Teldrive functions as a wrapper over your Telegram account, simplifying file access. However, users must adhere to the limitations imposed by the Telegram API. Teldrive is not responsible for any consequences arising from non-compliance with these API limits.You will be banned instantly if you misuse telegram API.

Visit https://teldrive-docs.pages.dev for setting up teldrive.

# Recognitions

<a href="https://trendshift.io/repositories/7568" target="_blank"><img src="https://trendshift.io/api/badge/repositories/7568" alt="divyam234%2Fteldrive | Trendshift" style="width: 250px; height: 55px;" width="250" height="55"/></a>

## Best Practices for Using Teldrive

### Dos:

- **Follow Limits:** Adhere to the limits imposed by Telegram servers to avoid account bans and automatic deletion of your channel.Your files will be removed from telegram servers if you try to abuse the service as most people have zero brains they will still do so good luck.
- **Responsible Storage:** Be mindful of the content you store on Telegram. Utilize storage efficiently and only keep data that serves a purpose.
  
### Don'ts:
- **Data Hoarding:** Avoid excessive data hoarding, as it not only violates Telegram's terms.
  
By following these guidelines, you contribute to the responsible and effective use of Telegram, maintaining a fair and equitable environment for all users.

## Contributing

Feel free to contribute to this project.See [CONTRIBUTING.md](CONTRIBUTING.md) for more information.

## Donate

If you like this project small contribution would be appreciated [Paypal](https://paypal.me/redux234).

## Star History

<a href="https://www.star-history.com/#tgdrive/teldrive&Date">
 <picture>
   <source media="(prefers-color-scheme: dark)" srcset="https://api.star-history.com/svg?repos=tgdrive/teldrive&type=Date&theme=dark" />
   <source media="(prefers-color-scheme: light)" srcset="https://api.star-history.com/svg?repos=tgdrive/teldrive&type=Date" />
   <img alt="Star History Chart" src="https://api.star-history.com/svg?repos=tgdrive/teldrive&type=Date" />
 </picture>
</a>

## Static API Key & Web UI Login Bypass

Teldrive supports a permanent Static API Key for third-party integrations and Web UI login bypass without needing to authenticate via Telegram.

### 1. Configuration (`config.toml`)
Add the following fields under the `[jwt]` section:
```toml
[jwt]
secret = "your-jwt-secret-key"
session-time = "30d"
api-key = "your-very-secure-static-api-key" # Define your static key here
api-key-user = 0 # Telegram User ID to map, or 0 to automatically use the first session in the database
```

### 2. API Usage
You can authenticate all Teldrive API requests (such as uploads, directory creation, file queries) using your Static API Key in one of the following ways:
- **Header:** `Authorization: Bearer your-very-secure-static-api-key`
- **Header:** `X-API-Key: your-very-secure-static-api-key`
- **Query Parameter:** `?token=your-very-secure-static-api-key`

This enables permanent API access for scripts and external applications (e.g., video and image uploads) that do not support standard cookie-based sessions.

### 3. HTML Streaming
To embed or stream videos and images directly in HTML without sending authorization headers, append the `?token=` parameter to the file URL:
```html
<!-- Embed an image -->
<img src="http://your-teldrive:8080/api/files/file-id/image.jpg?token=your-very-secure-static-api-key" />

<!-- Stream a video -->
<video controls>
  <source src="http://your-teldrive:8080/api/files/file-id/video.mp4?token=your-very-secure-static-api-key" type="video/mp4" />
</video>
```

### 4. Web UI Login Bypass
To log in to the Teldrive Web UI on any browser without performing a Telegram login, visit the following URL in your browser:
```
http://your-teldrive:8080/api/auth/static?key=your-very-secure-static-api-key
```
This endpoint will automatically verify the key, generate a valid session cookie, and redirect you to the Web UI dashboard logged in as the designated user.

