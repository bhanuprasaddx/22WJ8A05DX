<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>URL Shortener</title>
  <script src="https://cdn.jsdelivr.net/npm/react@18/umd/react.development.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/react-dom@18/umd/react-dom.development.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/@babel/standalone/babel.min.js"></script>
  <script src="https://cdn.tailwindcss.com"></script>
</head>
<body>
  <div id="root"></div>
  <script type="text/babel">
    const { useState } = React;

    const URLShortener = () => {
      const [urls, setUrls] = useState([{ longUrl: '', validity: '', shortcode: '' }]);
      const [shortenedUrls, setShortenedUrls] = useState([]);
      const [error, setError] = useState('');

      const handleInputChange = (index, event) => {
        const newUrls = [...urls];
        newUrls[index][event.target.name] = event.target.value;
        setUrls(newUrls);
      };

      const addUrlField = () => {
        if (urls.length < 5) setUrls([...urls, { longUrl: '', validity: '', shortcode: '' }]);
      };

      const validateInputs = () => {
        for (let url of urls) {
          if (!url.longUrl || !/^(ftp|http|https):\/\/[^ "]+$/.test(url.longUrl)) {
            setError('Please enter a valid URL');
            return false;
          }
          if (url.validity && isNaN(url.validity)) {
            setError('Validity must be a number');
            return false;
          }
        }
        setError('');
        return true;
      };

      const shortenUrl = async () => {
        if (!validateInputs()) return;
        const results = await Promise.all(urls.map(async (url) => {
          const response = await fetch('/api/shorten', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ longUrl: url.longUrl, validity: url.validity || 30, shortcode: url.shortcode }),
          });
          const data = await response.json();
          return { ...url, shortUrl: data.shortUrl, expiry: data.expiry, clicks: 0 };
        }));
        setShortenedUrls(results);
      };

      return (
        <div className="container mx-auto p-4">
          <h1 className="text-2xl font-bold mb-4">URL Shortener</h1>
          {error && <p className="text-red-500 mb-4">{error}</p>}
          {urls.map((url, index) => (
            <div key={index} className="mb-4">
              <input
                type="text"
                name="longUrl"
                value={url.longUrl}
                onChange={(e) => handleInputChange(index, e)}
                placeholder="Enter long URL"
                className="border p-2 mr-2"
              />
              <input
                type="text"
                name="validity"
                value={url.validity}
                onChange={(e) => handleInputChange(index, e)}
                placeholder="Validity (minutes, optional)"
                className="border p-2 mr-2"
              />
              <input
                type="text"
                name="shortcode"
                value={url.shortcode}
                onChange={(e) => handleInputChange(index, e)}
                placeholder="Shortcode (optional)"
                className="border p-2 mr-2"
              />
            </div>
          ))}
          <button onClick={addUrlField} className="bg-blue-500 text-white p-2 mr-2" disabled={urls.length >= 5}>
            Add URL
          </button>
          <button onClick={shortenUrl} className="bg-green-500 text-white p-2">
            Shorten URLs
          </button>
          <div className="mt-4">
            <h2 className="text-xl font-bold">Shortened URLs</h2>
            {shortenedUrls.map((url, index) => (
              <div key={index} className="mb-2">
                <p>Short URL: {url.shortUrl}, Expiry: {url.expiry}, Clicks: {url.clicks}</p>
              </div>
            ))}
          </div>
        </div>
      );
    };

    ReactDOM.render(<URLShortener />, document.getElementById('root'));
  </script>
</body>
</html>
