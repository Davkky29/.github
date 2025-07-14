npm install -g react-native-cli
npm install express mongoose stripe axios
// server.js
const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');
const app = express();

app.use(cors());
app.use(express.json());

mongoose.connect('mongodb://localhost/amazon-clone', { useNewUrlParser: true, useUnifiedTopology: true })
  .then(() => console.log('MongoDB connected'))
  .catch(err => console.log(err));

app.get('/', (req, res) => res.send('Amazon Clone API'));
app.listen(5000, () => console.log('Server running on port 5000'));
// models/User.js
const mongoose = require('mongoose');
const userSchema = new mongoose.Schema({
  email: { type: String, required: true, unique: true },
  password: { type: String, required: true },
  name: String,
});
module.exports = mongoose.model('User', userSchema);
// models/Product.js
const mongoose = require('mongoose');
const productSchema = new mongoose.Schema({
  name: String,
  price: Number,
  description: String,
  image: String,
  category: String,
});
module.exports = mongoose.model('Product', productSchema);
// routes/api.js
const express = require('express');
const router = express.Router();
const User = require('../models/User');
const Product = require('../models/Product');

// Register user
router.post('/register', async (req, res) => {
  const { email, password, name } = req.body;
  try {
    const user = new User({ email, password, name });
    await user.save();
    res.status(201).json({ message: 'User registered' });
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});

// Get products
router.get('/products', async (req, res) => {
  const products = await Product.find();
  res.json(products);
});

module.exports = router;
const apiRoutes = require('./routes/api');
app.use('/api', apiRoutes);
// routes/payment.js
const express = require('express');
const router = express.Router();
const stripe = require('stripe')('your-stripe-secret-key');

router.post('/create-payment-intent', async (req, res) => {
  const { amount } = req.body;
  try {
    const paymentIntent = await stripe.paymentIntents.create({
      amount: amount * 100, // Amount in cents
      currency: 'usd',
    });
    res.json({ clientSecret: paymentIntent.client_secret });
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});

module.exports = router;
const paymentRoutes = require('./routes/payment');
app.use('/api/payment', paymentRoutes);
npx react-native init AmazonClone
cd AmazonClone
npm install @react-navigation/native @react-navigation/stack axios stripe/stripe-react-native
// App.js
import { NavigationContainer } from '@react-navigation/native';
import { createStackNavigator } from '@react-navigation/stack';
import HomeScreen from './screens/HomeScreen';
import ProductScreen from './screens/ProductScreen';
import CartScreen from './screens/CartScreen';
import LoginScreen from './screens/LoginScreen';

const Stack = createStackNavigator();

export default function App() {
  return (
    <NavigationContainer>
      <Stack.Navigator>
        <Stack.Screen name="Login" component={LoginScreen} />
        <Stack.Screen name="Home" component={HomeScreen} />
        <Stack.Screen name="Product" component={ProductScreen} />
        <Stack.Screen name="Cart" component={CartScreen} />
      </Stack.Navigator>
    </NavigationContainer>
  );
}
// screens/LoginScreen.js
import React, { useState } from 'react';
import { View, TextInput, Button, Text } from 'react-native';
import axios from 'axios';

const LoginScreen = ({ navigation }) => {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');

  const handleRegister = async () => {
    try {
      await axios.post('http://your-server-ip:5000/api/register', { email, password });
      navigation.navigate('Home');
    } catch (error) {
      console.error(error);
    }
  };

  return (
    <View style={{ padding: 20 }}>
      <TextInput
        placeholder="Email"
        value={email}
        onChangeText={setEmail}
        style={{ borderWidth: 1, marginBottom: 10, padding: 10 }}
      />
      <TextInput
        placeholder="Password"
        value={password}
        onChangeText={setPassword}
        secureTextEntry
        style={{ borderWidth: 1, marginBottom: 10, padding: 10 }}
      />
      <Button title="Register" onPress={handleRegister} />
    </View>
  );
};

export default LoginScreen;
// screens/HomeScreen.js
import React, { useState, useEffect } from 'react';
import { View, Text, FlatList, TouchableOpacity } from 'react-native';
import axios from 'axios';

const HomeScreen = ({ navigation }) => {
  const [products, setProducts] = useState([]);

  useEffect(() => {
    axios.get('http://your-server-ip:5000/api/products')
      .then(response => setProducts(response.data))
      .catch(error => console.error(error));
  }, []);

  return (
    <View style={{ padding: 20 }}>
      <FlatList
        data={products}
        keyExtractor={item => item._id}
        renderItem={({ item }) => (
          <TouchableOpacity onPress={() => navigation.navigate('Product', { product: item })}>
            <Text>{item.name} - ${item.price}</Text>
          </TouchableOpacity>
        )}
      />
    </View>
  );
};

export default HomeScreen;
// screens/CartScreen.js
import React from 'react';
import { View, Text, Button } from 'react-native';
import { useStripe } from '@stripe/stripe-react-native';
import axios from 'axios';

const CartScreen = () => {
  const { initPaymentSheet, presentPaymentSheet } = useStripe();

  const handleCheckout = async () => {
    try {
      const response = await axios.post('http://your-server-ip:5000/api/payment/create-payment-intent', {
        amount: 1000, // $10.00
      });
      const { clientSecret } = response.data;

      await initPaymentSheet({ paymentIntentClientSecret: clientSecret });
      const { error } = await presentPaymentSheet();

      if (error) {
        console.log('Payment failed:', error.message);
      } else {
        console.log('Payment successful');
      }
    } catch (error) {
      console.error(error);
    }
  };

  return (
    <View style={{ padding: 20 }}>
      <Text>Cart Total: $10.00</Text>
      <Button title="Checkout" onPress={handleCheckout} />
    </View>
  );
};

export default CartScreen;
cd AmazonClone
npx react-native run-android --variant=release
