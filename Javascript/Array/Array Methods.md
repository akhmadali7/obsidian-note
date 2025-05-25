#js 
## Filter
## Map
```jsx
const AuthProvider = ({children}) => {  
    const [isLoggedIn, setIsLoggedIn] = useState(true)  
    return (  
        <AuthContext.Provider value={{isLoggedIn, setIsLoggedIn}}>  
            {isLoggedIn ? children :  
                <div className="container flex flex-col items-center justify-center min-h-screen">  
                    <p>Belum login lu </p>  
                    <button onClick={() => {  
                        setIsLoggedIn(prevState => !prevState)  
                    }}>Login deh  
                    </button>  
                </div>  
            }  
        </AuthContext.Provider>  
    )  
}
```
## Find
## ForEach
## Some
## Every
## Reduce

## Includes
