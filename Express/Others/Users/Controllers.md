``` javascript
/** @namefile  user.controller */
import asyncHandler from 'express-async-handler';  
import User from '../models/user.model.js';  
import generateToken from '../utils/token.generator.js';

const registerUser = asyncHandler(async (req, res) => {  
        const {name, email, password} = req.body  
        const userExist = await User.findOne({email})  
  
        if (userExist) {  
            res.status(400)  
            throw new Error('User already exist')  
        }  
        const user = await User.create({  
            name,  
            email,  
            password  
        })  
  
        if (user) {  
            generateToken(res, user._id)  
            res.status(201).json({  
                _id: user._id,  
                name: user.name,  
                email: user.email,  
                password: user.password  
            })  
        } else {  
            res.status(400)  
            throw new Error('Invalid User Data')  
        }    })
```
