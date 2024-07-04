# concept-cookie-token-csrf-jwt
1. so implement csrf token - also refer to the link https://github.com/saurabh021184/go/tree/main/4.1%20Handle%20CSRF%20security%20threat%20middleware
2. so as the link already explained what we are trying to achieve there are few more points i want to share as what is cookie vs token vs stateless and stateful and AWS WAF
3. SO the first solution implement csrf token using gin/csrf package
4. ```go
       package main
    
      import (
      	"context"
      	"net/http"
      	"os"
      	"os/signal"
      	"sman-services/config"
      	"sman-services/internal/handlers"
      	"sman-services/pkg/database/postgres"
      	"sman-services/pkg/logger"
      	"sman-services/pkg/utils"
      	"time"
      
      	"github.com/gin-gonic/gin"
      	"github.com/gin-contrib/sessions"
          "github.com/gin-contrib/sessions/cookie"
      //    "github.com/gin-gonic/gin"
          "github.com/utrack/gin-csrf"
      	"fmt"
      )
      
      var (
      	// Logger
      	log = logger.Log
      )
      
      func main() {
      	// Set mode debug/release
      	gin.SetMode(utils.GetGinMode(config.DebugLevel))
      
      	// Initialize Server
      	router := gin.New()
      	router.UseH2C = true
      	store := cookie.NewStore([]byte("secret"))
          router.Use(sessions.Sessions("mysession", store))
      	store.Options(sessions.Options{MaxAge: 60})
          router.Use(csrf.Middleware(csrf.Options{
                  Secret: "secret123",
                  ErrorFunc: func(c *gin.Context) {
                          c.String(400, "CSRF token mismatch")
                          c.Abort()
                  },
      //              IgnoreMethods: []string{"POST"},
          }))
      
      	router.GET("/protected", func(c *gin.Context) {
              //c.String(200, csrf.GetToken(c))
      		fmt.Println("c.Request.Proto ", c.Request.Proto)
      		token := csrf.GetToken(c)
      		cookie := c.Writer.Header().Get("Set-Cookie")
      		c.JSON(200, gin.H{
      			"X-CSRF-Token": token, //csrf.GetToken(c),
      			"Cookie":       cookie,
      		})
          })
      	router.POST("/protected",  func(c *gin.Context) {
                      c.String(200, "CSRF token is valid")
              })
  
   ```
5. Key points:
   -  ```go
      // Initialize Server
    	router := gin.New()
    	router.UseH2C = true
    	store := cookie.NewStore([]byte("secret"))
      ```
      Here we are creating a cookie store .. ( so in the backend it will start sroking cookies occupying space)
   -  ```go
      router.Use(sessions.Sessions("mysession", store))
    	store.Options(sessions.Options{MaxAge: 60})
      ```
      Then we are using that cookie store to set a session and also with store.Options we can set invalidation tikme of the cookie/session
   -  ```go
              router.Use(csrf.Middleware(csrf.Options{
                    Secret: "secret123",
                    ErrorFunc: func(c *gin.Context) {
                            c.String(400, "CSRF token mismatch")
                            c.Abort()
                    },
        //              IgnoreMethods: []string{"POST"},
            }))

      ```
      Then we are creatung a csrf middleware
   -  ```go
              router.GET("/protected", func(c *gin.Context) {
              //c.String(200, csrf.GetToken(c))
      		fmt.Println("c.Request.Proto ", c.Request.Proto)
      		token := csrf.GetToken(c)
      		cookie := c.Writer.Header().Get("Set-Cookie")
      		c.JSON(200, gin.H{
      			"X-CSRF-Token": token, //csrf.GetToken(c),
      			"Cookie":       cookie,
      		})
          })


      ```
      Then we are creating a GET API here we are getting csrf token csrf.GetToken and the setting the cookie c.Writer.Header().Get("Set-Cookie") and passing both of them in API response.. this API will be called initially the moment client logs in and .both these values will then be used by client to pass in next set of APIs
