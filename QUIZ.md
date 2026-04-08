## Q1
B

## Q2
`400 Bad Request` - this occurs when the request is formatted incorrectly somehow. For example, if we have a `POST /api/scores` with `"score": "ten"`, this would result in a `400 Bad Request` error because the score should be formatted as a number. 

`401 Unauthorized` - this occurs when a request does not have the proper content to authenticate it. For example, if we have a `POST /api/scores` without the `Authorization` header, we would get a `401 Unauthorized` error because the request lacks the `Bearer` token. 

`404 Not Found` - this occurs when we make a request to a page that is not present. For example, if we try to access a page like `quizblitz/account`, we would get a `404 Not Found` error because there is no account page. 

## Q3
The problem is that `Score.find()` is asynchronous, but the above code does not wait for the response. This means that the `message: 'done'` gets executed first while the code should be waiting for the response from `Score.find()`. Here is the corrected code:

```JS
app.get('/api/scores', async (req, res) => {
    try {
        const scores = await Score.find()
        .sort({ score: -1 })
        .limit(10)
        res.json(scores)
    } catch (error) {
        console.error('Error fetching scores:', error.message)
        res.status(500).json({ error: 'Failed to fetch scores' })
    }
})
```

This way we actually wait for the response and get scores returned. 

## Q4
B 

## Q5
One advantage of storing the JWT as a cookie is that cookies are automatically included in all requests to their respective domain, so the JWT doesn't need to be manually added to requests. 

One advantage of sticking with the `Authorization` header is that it works with all browsers, including mobile browsers, as well as mobile apps. It turns out that mobile browsers handle cookies in different ways and the same cannot be said for the cookie approach. 

Hence, the `Authorization` header is more appropriate for mobile-accessible games like Quizblitz, since it is a more universal approach for different devices. 