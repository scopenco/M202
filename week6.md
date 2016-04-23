# CHAPTER 6: SECURITY, AUTHENTICATION AND AUTHORIZATION

# Videos

* [Securing MongoDB overview](https://www.youtube.com/watch?v=ZV15RYBHsNQ)
* [Securing MongoDB step by step](https://www.youtube.com/watch?v=-Kngi_avFE0)
* [Locking down the network](https://www.youtube.com/watch?v=vagtFWxWt_k)
* [Evolution of native auth in MongoDB](https://www.youtube.com/watch?v=G3hB8V-4cYQ)
* [Native Auth in MongoDB 2.6](https://www.youtube.com/watch?v=CQGZO3x4ICM)
* [Authentication in sharded clusters: Databases](https://www.youtube.com/watch?v=nFkzp2b1MLo)
* [Authentication in sharded clusters: Collections](https://www.youtube.com/watch?v=SwLn4rP2AAE)
* [Authentication in sharded clusters: Users](https://www.youtube.com/watch?v=JYH5ExoUrig)
* [Authorization: Concepts](https://www.youtube.com/watch?v=yLQf6KuwudM)
* [Authorization: Roles](https://www.youtube.com/watch?v=fOIElCEo4eg)
* [Authorization: Demo](https://www.youtube.com/watch?v=HYiR_ygEJbI)
* [Using SSL with MongoDB](https://www.youtube.com/watch?v=1SMocMx7hVI)
* [Using SSL with MongoDB 2.6](https://www.youtube.com/watch?v=OUwmjtVJ3dw)
* [SSL with MongoDB demo](https://www.youtube.com/watch?v=mK9OsoOr3ks)
* [SSL with MongoDB caveats](https://www.youtube.com/watch?v=Gudgmsfubh4)

# Homeworks
## Homework: 6.1 Security Best Practices 
### Question
Which of the following are recommended best practices for MongoDB deployments? Check all that apply.

### Answer
```
- Don't limit the source IP addresses for incoming connections
+ Use role-based access control
+ Encrypt all communication with TLS/SSL
```

## Homework: 6.2: SSL
### Question
In mongoDB, which of the following are true of SSL (when SSL is required across the entire cluster, through the "requireSSL" mode)? Check all that apply:

### Answer
```
+ SSL encrypts all communication between mongodb servers
+ SSL compresses data communicated between servers
+ SSL helps to prevent man-in-the-middle attacks
```
