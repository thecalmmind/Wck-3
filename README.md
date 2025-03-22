# Wck-3
import { NavigationContainer } from '@react-navigation/native';
import { createNativeStackNavigator } from '@react-navigation/native-stack';
import { StyleSheet } from 'react-native';
import { SafeAreaProvider } from "react-native-safe-area-context"
import { Toaster } from 'sonner-native';
import HomeScreen from "./screens/HomeScreen"
import OrderScreen from "./screens/OrderScreen"
import MembershipScreen from "./screens/MembershipScreen"
import SellerDashboardScreen from "./screens/SellerDashboardScreen"
import ChatScreen from "./screens/ChatScreen"

const Stack = createNativeStackNavigator();

function RootStack() {
  return (    <Stack.Navigator screenOptions={{
      headerShown: false
    }}>
      <Stack.Screen name="Home" component={HomeScreen} />
      <Stack.Screen name="Order" component={OrderScreen} />
      <Stack.Screen name="Membership" component={MembershipScreen} />      <Stack.Screen name="SellerDashboard" component={SellerDashboardScreen} />
      <Stack.Screen name="Chat" component={ChatScreen} />
    </Stack.Navigator>
  );
}

export default function App() {
  return (
    <SafeAreaProvider style={styles.container}>
      <Toaster />
      <NavigationContainer>
        <RootStack />
      </NavigationContainer>
    </SafeAreaProvider>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1
  }
});
import React from 'react';
import { View, Text, StyleSheet, Modal, TouchableOpacity } from 'react-native';
import { MaterialIcons } from '@expo/vector-icons';

export default function FilterModal({ visible, onClose, sortBy, setSortBy }) {
  return (
    <Modal
      visible={visible}
      transparent
      animationType="slide"
      onRequestClose={onClose}
    >
      <View style={styles.modalContainer}>
        <View style={styles.modalContent}>
          <View style={styles.header}>
            <Text style={styles.title}>Sort & Filter</Text>
            <TouchableOpacity onPress={onClose}>
              <MaterialIcons name="close" size={24} color="#333" />
            </TouchableOpacity>
          </View>

          <Text style={styles.sectionTitle}>Sort by</Text>
          
          <TouchableOpacity
            style={[styles.option, sortBy === 'distance' && styles.selectedOption]}
            onPress={() => {
              setSortBy('distance');
              onClose();
            }}
          >
            <MaterialIcons 
              name="location-on" 
              size={20} 
              color={sortBy === 'distance' ? '#FF4D4D' : '#666'} 
            />
            <Text style={[
              styles.optionText,
              sortBy === 'distance' && styles.selectedOptionText
            ]}>
              Distance
            </Text>
          </TouchableOpacity>

          <TouchableOpacity
            style={[styles.option, sortBy === 'rating' && styles.selectedOption]}
            onPress={() => {
              setSortBy('rating');
              onClose();
            }}
          >
            <MaterialIcons 
              name="star" 
              size={20} 
              color={sortBy === 'rating' ? '#FF4D4D' : '#666'} 
            />
            <Text style={[
              styles.optionText,
              sortBy === 'rating' && styles.selectedOptionText
            ]}>
              Rating
            </Text>
          </TouchableOpacity>

          <TouchableOpacity
            style={[styles.option, sortBy === 'price' && styles.selectedOption]}
            onPress={() => {
              setSortBy('price');
              onClose();
            }}
          >
            <MaterialIcons 
              name="monetization-on" 
              size={20} 
              color={sortBy === 'price' ? '#FF4D4D' : '#666'} 
            />
            <Text style={[
              styles.optionText,
              sortBy === 'price' && styles.selectedOptionText
            ]}>
              Price
            </Text>
          </TouchableOpacity>
        </View>
      </View>
    </Modal>
  );
}

const styles = StyleSheet.create({
  modalContainer: {
    flex: 1,
    backgroundColor: 'rgba(0, 0, 0, 0.5)',
    justifyContent: 'flex-end',
  },
  modalContent: {
    backgroundColor: '#fff',
    borderTopLeftRadius: 20,
    borderTopRightRadius: 20,
    padding: 20,
    minHeight: 300,
  },
  header: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    marginBottom: 20,
  },
  title: {
    fontSize: 18,
    fontWeight: 'bold',
  },
  sectionTitle: {
    fontSize: 16,
    fontWeight: '600',
    marginBottom: 12,
    color: '#666',
  },
  option: {
    flexDirection: 'row',
    alignItems: 'center',
    padding: 12,
    borderRadius: 8,
    marginBottom: 8,
  },
  selectedOption: {
    backgroundColor: '#FFF0F0',
  },
  optionText: {
    marginLeft: 12,
    fontSize: 16,
    color: '#333',
  },
  selectedOptionText: {
    color: '#FF4D4D',
    fontWeight: '500',
  },
});
import React, { useState, useEffect } from 'react';
import { View, Text, StyleSheet, FlatList, Image, TouchableOpacity, RefreshControl, TextInput, Modal } from 'react-native';
import * as Location from 'expo-location';
import { MaterialIcons, Ionicons } from '@expo/vector-icons';
import { useNavigation } from '@react-navigation/native';
import { SafeAreaView } from 'react-native-safe-area-context';

const FOOD_CATEGORIES = ['All', 'North Indian', 'South Indian', 'Chinese', 'Italian', 'Desserts', 'Beverages'];

const MOCK_POSTS = [
  {
    id: '1',
    username: 'TastyBites',
    userImage: 'https://api.a0.dev/assets/image?text=chef+profile+picture&aspect=1:1',
    foodImage: 'https://api.a0.dev/assets/image?text=delicious+biryani+food+photography&aspect=4:5',
    description: 'Homemade Biryani 😋 Available now!',
    price: '₹199',
    isOnline: true,
    distance: '2.3km',
    category: 'North Indian',
    rating: 4.5,
    likes: 124,
  },
  {
    id: '2',
    username: 'SweetTreats',
    userImage: 'https://api.a0.dev/assets/image?text=baker+profile+picture&aspect=1:1',
    foodImage: 'https://api.a0.dev/assets/image?text=chocolate+cake+dessert+photography&aspect=4:5',
    description: 'Fresh Chocolate Cake 🎂',
    price: '₹399',
    isOnline: true,
    distance: '3.1km',
    category: 'Desserts',
    rating: 4.8,
    likes: 89,
  },
  {
    id: '1',
    username: 'TastyBites',
    userImage: 'https://api.a0.dev/assets/image?text=chef+profile+picture&aspect=1:1',
    foodImage: 'https://api.a0.dev/assets/image?text=delicious+biryani+food+photography&aspect=4:5',
    description: 'Homemade Biryani 😋 Available now!',
    price: '₹199',
    isOnline: true,
    distance: '2.3km',
  },
  {
    id: '2',
    username: 'SweetTreats',
    userImage: 'https://api.a0.dev/assets/image?text=baker+profile+picture&aspect=1:1',
    foodImage: 'https://api.a0.dev/assets/image?text=chocolate+cake+dessert+photography&aspect=4:5',
    description: 'Fresh Chocolate Cake 🎂',
    price: '₹399',
    isOnline: true,
    distance: '3.1km',
  },
];

export default function HomeScreen() {
  const navigation = useNavigation();
  const [refreshing, setRefreshing] = useState(false);
  const [searchQuery, setSearchQuery] = useState('');
  const [selectedCategory, setSelectedCategory] = useState('All');
  const [showFilters, setShowFilters] = useState(false);
  const [location, setLocation] = useState(null);
  const [sortBy, setSortBy] = useState('distance'); // 'distance', 'rating', 'price'

  useEffect(() => {
    (async () => {
      let { status } = await Location.requestForegroundPermissionsAsync();
      if (status === 'granted') {
        let location = await Location.getCurrentPositionAsync({});
        setLocation(location);
      }
    })();
  }, []);

  const filteredPosts = MOCK_POSTS.filter(post => {
    const matchesSearch = post.description.toLowerCase().includes(searchQuery.toLowerCase()) ||
                         post.username.toLowerCase().includes(searchQuery.toLowerCase());
    const matchesCategory = selectedCategory === 'All' || post.category === selectedCategory;
    return matchesSearch && matchesCategory;
  }).sort((a, b) => {
    if (sortBy === 'distance') return parseFloat(a.distance) - parseFloat(b.distance);
    if (sortBy === 'rating') return b.rating - a.rating;
    if (sortBy === 'price') return parseFloat(a.price.replace('₹', '')) - parseFloat(b.price.replace('₹', ''));
    return 0;
  });

  const onRefresh = React.useCallback(() => {
    setRefreshing(true);
    // Simulate refresh
    setTimeout(() => {
      setRefreshing(false);
    }, 2000);
  }, []);  const renderPost = ({ item }) => (
    <View style={styles.postContainer}>
      <View style={styles.postHeader}>
        <View style={styles.userInfo}>
          <Image source={{ uri: item.userImage }} style={styles.userImage} />
          <Text style={styles.username}>{item.username}</Text>
          {item.isOnline && <View style={styles.onlineIndicator} />}
        </View>
        <View style={styles.distanceContainer}>
          <MaterialIcons name="location-on" size={16} color="#666" />
          <Text style={styles.distanceText}>{item.distance}</Text>
        </View>
      </View>

      <Image source={{ uri: item.foodImage }} style={styles.foodImage} />

      <View style={styles.postFooter}>
        <View style={styles.actionButtons}>
          <TouchableOpacity style={styles.likeButton}>
            <Ionicons name="heart-outline" size={24} color="#333" />
            <Text style={styles.likeCount}>{item.likes}</Text>
          </TouchableOpacity>
          <TouchableOpacity style={styles.commentButton}>
            <Ionicons name="chatbubble-outline" size={24} color="#333" />
          </TouchableOpacity>
          <TouchableOpacity style={styles.shareButton}>
            <Ionicons name="share-social-outline" size={24} color="#333" />
          </TouchableOpacity>
        </View>

        <View style={styles.postDetails}>
          <Text style={styles.description}>{item.description}</Text>
          <Text style={styles.categoryTag}>{item.category}</Text>
          <View style={styles.ratingContainer}>
            <MaterialIcons name="star" size={16} color="#FFD700" />
            <Text style={styles.rating}>{item.rating}</Text>
          </View>
          <Text style={styles.price}>{item.price}</Text>
          <TouchableOpacity 
            style={styles.orderButton}
            onPress={() => navigation.navigate('Order', { item })}
          >
            <Text style={styles.orderButtonText}>Order Now</Text>
          </TouchableOpacity>
        </View>
      </View>
    </View>
  );
    <View style={styles.postContainer}>
      <View style={styles.postHeader}>
        <View style={styles.userInfo}>
          <Image source={{ uri: item.userImage }} style={styles.userImage} />
          <Text style={styles.username}>{item.username}</Text>
          {item.isOnline && <View style={styles.onlineIndicator} />}
        </View>
        <View style={styles.distanceContainer}>
          <MaterialIcons name="location-on" size={16} color="#666" />
          <Text style={styles.distanceText}>{item.distance}</Text>
        </View>
      </View>

      <Image source={{ uri: item.foodImage }} style={styles.foodImage} />

      <View style={styles.postFooter}>
        <View style={styles.actionButtons}>
          <TouchableOpacity style={styles.likeButton}>
            <Ionicons name="heart-outline" size={24} color="#333" />
          </TouchableOpacity>
          <TouchableOpacity style={styles.commentButton}>
            <Ionicons name="chatbubble-outline" size={24} color="#333" />
          </TouchableOpacity>
        </View>

        <View style={styles.postDetails}>
          <Text style={styles.description}>{item.description}</Text>
          <Text style={styles.price}>{item.price}</Text>
          <TouchableOpacity 
            style={styles.orderButton}
            onPress={() => navigation.navigate('Order', { item })}
          >
            <Text style={styles.orderButtonText}>Order Now</Text>
          </TouchableOpacity>
        </View>
      </View>
    </View>
  );  return (
    <SafeAreaView style={styles.container}>
      <View style={styles.searchContainer}>
        <TextInput
          style={styles.searchInput}
          placeholder="Search foods or sellers..."
          value={searchQuery}
          onChangeText={setSearchQuery}
        />
        <TouchableOpacity onPress={() => setShowFilters(true)}>
          <MaterialIcons name="tune" size={24} color="#666" />
        </TouchableOpacity>
      </View>

      <ScrollView 
        horizontal 
        showsHorizontalScrollIndicator={false}
        style={styles.categoriesContainer}
      >
        {FOOD_CATEGORIES.map((category) => (
          <TouchableOpacity
            key={category}
            style={[
              styles.categoryButton,
              selectedCategory === category && styles.selectedCategory
            ]}
            onPress={() => setSelectedCategory(category)}
          >
            <Text style={[
              styles.categoryText,
              selectedCategory === category && styles.selectedCategoryText
            ]}>
              {category}
            </Text>
          </TouchableOpacity>
        ))}
      </ScrollView>      <View style={styles.header}>
        <Text style={styles.headerTitle}>WCK</Text>
        <View style={styles.headerButtons}>
          <TouchableOpacity 
            style={styles.sellerButton} 
            onPress={() => navigation.navigate('SellerDashboard')}
          >
            <MaterialIcons name="store" size={20} color="#FF4D4D" />
          </TouchableOpacity>
          <TouchableOpacity onPress={() => navigation.navigate('Membership')}>
            <Text style={styles.membershipButton}>Join ₹99/month</Text>
          </TouchableOpacity>
        </View>
      </View>

      <FlatList
        data={MOCK_POSTS}
        renderItem={renderPost}
        keyExtractor={item => item.id}
        showsVerticalScrollIndicator={false}
        refreshControl={
          <RefreshControl refreshing={refreshing} onRefresh={onRefresh} />
        }
      />
    </SafeAreaView>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff',
  },  header: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    padding: 16,
    borderBottomWidth: 1,
    borderBottomColor: '#eee',
  },
  headerButtons: {
    flexDirection: 'row',
    alignItems: 'center',
  },
  sellerButton: {
    marginRight: 16,
    padding: 8,
    borderRadius: 20,
    backgroundColor: '#fff',
    borderWidth: 1,
    borderColor: '#FF4D4D',
  },
  headerTitle: {
    fontSize: 24,
    fontWeight: 'bold',
    color: '#333',
  },
  membershipButton: {
    color: '#FF4D4D',
    fontWeight: '600',
  },
  postContainer: {
    marginBottom: 20,
  },
  postHeader: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    padding: 12,
  },
  userInfo: {
    flexDirection: 'row',
    alignItems: 'center',
  },
  userImage: {
    width: 32,
    height: 32,
    borderRadius: 16,
    marginRight: 8,
  },
  username: {
    fontWeight: '600',
    marginRight: 8,
  },
  onlineIndicator: {
    width: 8,
    height: 8,
    borderRadius: 4,
    backgroundColor: '#4CAF50',
  },
  distanceContainer: {
    flexDirection: 'row',
    alignItems: 'center',
  },
  distanceText: {
    color: '#666',
    marginLeft: 4,
  },
  foodImage: {
    width: '100%',
    height: 400,
    resizeMode: 'cover',
  },
  postFooter: {
    padding: 12,
  },
  actionButtons: {
    flexDirection: 'row',
    marginBottom: 8,
  },
  likeButton: {
    marginRight: 16,
  },
  postDetails: {
    marginTop: 8,
  },
  description: {
    fontSize: 14,
    marginBottom: 4,
  },
  price: {
    fontSize: 16,
    fontWeight: 'bold',
    color: '#FF4D4D',
    marginBottom: 8,
  },
  orderButton: {
    backgroundColor: '#FF4D4D',
    padding: 12,
    borderRadius: 8,
    alignItems: 'center',
  },
  orderButtonText: {
    color: '#fff',
    fontWeight: '600',
  },
});
import React, { useState } from 'react';
import { View, Text, StyleSheet, FlatList, TextInput, TouchableOpacity, Image, KeyboardAvoidingView, Platform } from 'react-native';
import { SafeAreaView } from 'react-native-safe-area-context';
import { MaterialIcons } from '@expo/vector-icons';
import { useNavigation, useRoute } from '@react-navigation/native';

const MOCK_MESSAGES = [
  {
    id: '1',
    sender: 'seller',
    message: 'Hello! Thank you for your order. I will start preparing it right away.',
    timestamp: '10:30 AM',
  },
  {
    id: '2',
    sender: 'customer',
    message: 'Great! How long will it take?',
    timestamp: '10:31 AM',
  },
  {
    id: '3',
    sender: 'seller',
    message: 'It will take about 20 minutes to prepare. Is that okay?',
    timestamp: '10:32 AM',
  },
];

export default function ChatScreen() {
  const navigation = useNavigation();
  const route = useRoute();
  const { seller } = route.params;
  const [message, setMessage] = useState('');

  const sendMessage = () => {
    if (message.trim()) {
      // In a real app, this would send to a backend
      setMessage('');
    }
  };

  const renderMessage = ({ item }) => (
    <View style={[
      styles.messageContainer,
      item.sender === 'customer' ? styles.customerMessage : styles.sellerMessage
    ]}>
      <Text style={styles.messageText}>{item.message}</Text>
      <Text style={styles.timestamp}>{item.timestamp}</Text>
    </View>
  );

  return (
    <SafeAreaView style={styles.container}>
      <View style={styles.header}>
        <TouchableOpacity onPress={() => navigation.goBack()}>
          <MaterialIcons name="arrow-back" size={24} color="#333" />
        </TouchableOpacity>
        <View style={styles.sellerInfo}>
          <Image source={{ uri: seller.userImage }} style={styles.sellerImage} />
          <View>
            <Text style={styles.sellerName}>{seller.username}</Text>
            {seller.isOnline && (
              <View style={styles.onlineContainer}>
                <View style={styles.onlineIndicator} />
                <Text style={styles.onlineText}>Online</Text>
              </View>
            )}
          </View>
        </View>
        <View style={{ width: 24 }} />
      </View>

      <FlatList
        data={MOCK_MESSAGES}
        renderItem={renderMessage}
        keyExtractor={item => item.id}
        contentContainerStyle={styles.messagesList}
        inverted={false}
      />

      <KeyboardAvoidingView
        behavior={Platform.OS === 'ios' ? 'padding' : 'height'}
        keyboardVerticalOffset={Platform.OS === 'ios' ? 90 : 0}
      >
        <View style={styles.inputContainer}>
          <TextInput
            style={styles.input}
            value={message}
            onChangeText={setMessage}
            placeholder="Type a message..."
            multiline
          />
          <TouchableOpacity 
            style={styles.sendButton} 
            onPress={sendMessage}
            disabled={!message.trim()}
          >
            <MaterialIcons 
              name="send" 
              size={24} 
              color={message.trim() ? '#FF4D4D' : '#999'} 
            />
          </TouchableOpacity>
        </View>
      </KeyboardAvoidingView>
    </SafeAreaView>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff',
  },
  header: {
    flexDirection: 'row',
    alignItems: 'center',
    padding: 16,
    borderBottomWidth: 1,
    borderBottomColor: '#eee',
  },
  sellerInfo: {
    flex: 1,
    flexDirection: 'row',
    alignItems: 'center',
    marginLeft: 16,
  },
  sellerImage: {
    width: 40,
    height: 40,
    borderRadius: 20,
    marginRight: 12,
  },
  sellerName: {
    fontSize: 16,
    fontWeight: '600',
  },
  onlineContainer: {
    flexDirection: 'row',
    alignItems: 'center',
    marginTop: 4,
  },
  onlineIndicator: {
    width: 8,
    height: 8,
    borderRadius: 4,
    backgroundColor: '#4CAF50',
    marginRight: 4,
  },
  onlineText: {
    fontSize: 12,
    color: '#4CAF50',
  },
  messagesList: {
    padding: 16,
  },
  messageContainer: {
    maxWidth: '80%',
    padding: 12,
    borderRadius: 16,
    marginBottom: 12,
  },
  customerMessage: {
    backgroundColor: '#FF4D4D',
    alignSelf: 'flex-end',
    borderBottomRightRadius: 4,
  },
  sellerMessage: {
    backgroundColor: '#f0f0f0',
    alignSelf: 'flex-start',
    borderBottomLeftRadius: 4,
  },
  messageText: {
    fontSize: 16,
    color: '#333',
  },
  timestamp: {
    fontSize: 12,
    color: '#666',
    marginTop: 4,
    alignSelf: 'flex-end',
  },
  inputContainer: {
    flexDirection: 'row',
    padding: 16,
    borderTopWidth: 1,
    borderTopColor: '#eee',
    alignItems: 'flex-end',
  },
  input: {
    flex: 1,
    backgroundColor: '#f8f8f8',
    borderRadius: 20,
    paddingHorizontal: 16,
    paddingVertical: 8,
    maxHeight: 100,
    marginRight: 8,
  },
  sendButton: {
    width: 40,
    height: 40,
    borderRadius: 20,
    alignItems: 'center',
    justifyContent: 'center',
  },
});
import React from 'react';
import { View, Text, StyleSheet, TouchableOpacity, Image, ScrollView } from 'react-native';
import { SafeAreaView } from 'react-native-safe-area-context';
import { MaterialIcons } from '@expo/vector-icons';
import { useNavigation, useRoute } from '@react-navigation/native';
import { toast } from 'sonner-native';

export default function OrderScreen() {
  const navigation = useNavigation();
  const route = useRoute();
  const { item } = route.params;

  const handleOrder = () => {
    toast.success('Order placed successfully!');
    navigation.navigate('Home');
  };

  return (
    <SafeAreaView style={styles.container}>
      <View style={styles.header}>
        <TouchableOpacity onPress={() => navigation.navigate('Home')}>
          <MaterialIcons name="arrow-back" size={24} color="#333" />
        </TouchableOpacity>
        <Text style={styles.headerTitle}>Order Details</Text>
        <View style={{ width: 24 }} />
      </View>

      <ScrollView>
        <Image source={{ uri: item.foodImage }} style={styles.foodImage} />
        
        <View style={styles.content}>
          <View style={styles.sellerInfo}>
            <Image source={{ uri: item.userImage }} style={styles.sellerImage} />
            <View>
              <Text style={styles.sellerName}>{item.username}</Text>
              <View style={styles.distanceContainer}>
                <MaterialIcons name="location-on" size={16} color="#666" />
                <Text style={styles.distanceText}>{item.distance}</Text>
              </View>
            </View>
            <View style={styles.onlineStatus}>
              <View style={styles.onlineIndicator} />
              <Text style={styles.onlineText}>Online Now</Text>
            </View>
          </View>

          <Text style={styles.description}>{item.description}</Text>
          <Text style={styles.price}>{item.price}</Text>

          <View style={styles.deliveryInfo}>
            <Text style={styles.deliveryTitle}>Estimated Delivery Time</Text>
            <Text style={styles.deliveryTime}>20-30 minutes</Text>
          </View>          <View style={styles.buttonContainer}>
            <TouchableOpacity 
              style={styles.chatButton} 
              onPress={() => navigation.navigate('Chat', { seller: item })}
            >
              <MaterialIcons name="chat" size={20} color="#FF4D4D" />
              <Text style={styles.chatButtonText}>Chat with Seller</Text>
            </TouchableOpacity>

            <TouchableOpacity style={styles.orderButton} onPress={handleOrder}>
              <Text style={styles.orderButtonText}>Confirm Order</Text>
            </TouchableOpacity>
          </View>
        </View>
      </ScrollView>
    </SafeAreaView>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff',
  },
  header: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    padding: 16,
    borderBottomWidth: 1,
    borderBottomColor: '#eee',
  },
  headerTitle: {
    fontSize: 18,
    fontWeight: '600',
  },
  foodImage: {
    width: '100%',
    height: 300,
    resizeMode: 'cover',
  },
  content: {
    padding: 16,
  },
  sellerInfo: {
    flexDirection: 'row',
    alignItems: 'center',
    marginBottom: 20,
  },
  sellerImage: {
    width: 50,
    height: 50,
    borderRadius: 25,
    marginRight: 12,
  },
  sellerName: {
    fontSize: 16,
    fontWeight: '600',
    marginBottom: 4,
  },
  distanceContainer: {
    flexDirection: 'row',
    alignItems: 'center',
  },
  distanceText: {
    color: '#666',
    marginLeft: 4,
  },
  onlineStatus: {
    flexDirection: 'row',
    alignItems: 'center',
    marginLeft: 'auto',
  },
  onlineIndicator: {
    width: 8,
    height: 8,
    borderRadius: 4,
    backgroundColor: '#4CAF50',
    marginRight: 6,
  },
  onlineText: {
    color: '#4CAF50',
    fontSize: 12,
  },
  description: {
    fontSize: 16,
    marginBottom: 8,
  },
  price: {
    fontSize: 24,
    fontWeight: 'bold',
    color: '#FF4D4D',
    marginBottom: 20,
  },
  deliveryInfo: {
    backgroundColor: '#f8f8f8',
    padding: 16,
    borderRadius: 8,
    marginBottom: 20,
  },
  deliveryTitle: {
    fontSize: 14,
    color: '#666',
    marginBottom: 4,
  },
  deliveryTime: {
    fontSize: 16,
    fontWeight: '600',
  },  buttonContainer: {
    gap: 12,
  },
  chatButton: {
    flexDirection: 'row',
    alignItems: 'center',
    justifyContent: 'center',
    padding: 16,
    borderRadius: 8,
    backgroundColor: '#fff',
    borderWidth: 1,
    borderColor: '#FF4D4D',
  },
  chatButtonText: {
    color: '#FF4D4D',
    fontSize: 16,
    fontWeight: '600',
    marginLeft: 8,
  },
  orderButton: {
    backgroundColor: '#FF4D4D',
    padding: 16,
    borderRadius: 8,
    alignItems: 'center',
  },
  orderButtonText: {
    color: '#fff',
    fontSize: 16,
    fontWeight: '600',
  },
});
import React, { useState } from 'react';
import { View, Text, StyleSheet, ScrollView, TouchableOpacity, Image, FlatList } from 'react-native';
import { SafeAreaView } from 'react-native-safe-area-context';
import { MaterialIcons, Ionicons } from '@expo/vector-icons';
import { useNavigation } from '@react-navigation/native';

const MOCK_ORDERS = [
  {
    id: '1',
    customerName: 'Rahul Kumar',
    item: 'Biryani',
    price: '₹199',
    status: 'pending',
    time: '10 mins ago',
  },
  {
    id: '2',
    customerName: 'Priya Singh',
    item: 'Butter Chicken',
    price: '₹299',
    status: 'completed',
    time: '1 hour ago',
  },
];

const MOCK_MENU_ITEMS = [
  {
    id: '1',
    name: 'Chicken Biryani',
    price: '₹199',
    image: 'https://api.a0.dev/assets/image?text=biryani+food+photography&aspect=1:1',
    available: true,
  },
  {
    id: '2',
    name: 'Butter Chicken',
    price: '₹299',
    image: 'https://api.a0.dev/assets/image?text=butter+chicken+food+photography&aspect=1:1',
    available: true,
  },
];

export default function SellerDashboardScreen() {
  const navigation = useNavigation();
  const [activeTab, setActiveTab] = useState('orders');

  const renderOrder = ({ item }) => (
    <View style={styles.orderCard}>
      <View style={styles.orderHeader}>
        <Text style={styles.customerName}>{item.customerName}</Text>
        <Text style={[
          styles.orderStatus,
          { color: item.status === 'completed' ? '#4CAF50' : '#FF9800' }
        ]}>
          {item.status.charAt(0).toUpperCase() + item.status.slice(1)}
        </Text>
      </View>
      <View style={styles.orderDetails}>
        <Text style={styles.itemName}>{item.item}</Text>
        <Text style={styles.price}>{item.price}</Text>
      </View>
      <Text style={styles.orderTime}>{item.time}</Text>
      {item.status === 'pending' && (
        <View style={styles.actionButtons}>
          <TouchableOpacity style={[styles.actionButton, styles.acceptButton]}>
            <Text style={styles.actionButtonText}>Accept</Text>
          </TouchableOpacity>
          <TouchableOpacity style={[styles.actionButton, styles.rejectButton]}>
            <Text style={styles.actionButtonText}>Reject</Text>
          </TouchableOpacity>
        </View>
      )}
    </View>
  );

  const renderMenuItem = ({ item }) => (
    <View style={styles.menuItem}>
      <Image source={{ uri: item.image }} style={styles.menuItemImage} />
      <View style={styles.menuItemDetails}>
        <Text style={styles.menuItemName}>{item.name}</Text>
        <Text style={styles.menuItemPrice}>{item.price}</Text>
      </View>
      <TouchableOpacity 
        style={[styles.availabilityButton, 
          { backgroundColor: item.available ? '#4CAF50' : '#666' }
        ]}
      >
        <Text style={styles.availabilityButtonText}>
          {item.available ? 'Available' : 'Sold Out'}
        </Text>
      </TouchableOpacity>
    </View>
  );

  return (
    <SafeAreaView style={styles.container}>
      <View style={styles.header}>
        <Text style={styles.headerTitle}>Seller Dashboard</Text>
        <TouchableOpacity onPress={() => navigation.navigate('Home')}>
          <MaterialIcons name="exit-to-app" size={24} color="#333" />
        </TouchableOpacity>
      </View>

      <View style={styles.statsContainer}>
        <View style={styles.statCard}>
          <Text style={styles.statValue}>₹2,459</Text>
          <Text style={styles.statLabel}>Today's Earnings</Text>
        </View>
        <View style={styles.statCard}>
          <Text style={styles.statValue}>12</Text>
          <Text style={styles.statLabel}>Orders Today</Text>
        </View>
        <View style={styles.statCard}>
          <Text style={styles.statValue}>4.8 ★</Text>
          <Text style={styles.statLabel}>Rating</Text>
        </View>
      </View>

      <View style={styles.tabContainer}>
        <TouchableOpacity 
          style={[styles.tab, activeTab === 'orders' && styles.activeTab]}
          onPress={() => setActiveTab('orders')}
        >
          <Text style={[styles.tabText, activeTab === 'orders' && styles.activeTabText]}>
            Orders
          </Text>
        </TouchableOpacity>
        <TouchableOpacity 
          style={[styles.tab, activeTab === 'menu' && styles.activeTab]}
          onPress={() => setActiveTab('menu')}
        >
          <Text style={[styles.tabText, activeTab === 'menu' && styles.activeTabText]}>
            Menu Items
          </Text>
        </TouchableOpacity>
      </View>

      {activeTab === 'orders' ? (
        <FlatList
          data={MOCK_ORDERS}
          renderItem={renderOrder}
          keyExtractor={item => item.id}
          contentContainerStyle={styles.ordersList}
        />
      ) : (
        <View style={styles.menuContainer}>
          <TouchableOpacity style={styles.addItemButton}>
            <Ionicons name="add-circle-outline" size={24} color="#fff" />
            <Text style={styles.addItemButtonText}>Add New Item</Text>
          </TouchableOpacity>
          <FlatList
            data={MOCK_MENU_ITEMS}
            renderItem={renderMenuItem}
            keyExtractor={item => item.id}
            contentContainerStyle={styles.menuList}
          />
        </View>
      )}
    </SafeAreaView>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff',
  },
  header: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    padding: 16,
    borderBottomWidth: 1,
    borderBottomColor: '#eee',
  },
  headerTitle: {
    fontSize: 20,
    fontWeight: 'bold',
  },
  statsContainer: {
    flexDirection: 'row',
    padding: 16,
    justifyContent: 'space-between',
  },
  statCard: {
    backgroundColor: '#f8f8f8',
    padding: 16,
    borderRadius: 8,
    alignItems: 'center',
    flex: 1,
    marginHorizontal: 4,
  },
  statValue: {
    fontSize: 18,
    fontWeight: 'bold',
    color: '#FF4D4D',
  },
  statLabel: {
    fontSize: 12,
    color: '#666',
    marginTop: 4,
  },
  tabContainer: {
    flexDirection: 'row',
    padding: 16,
    borderBottomWidth: 1,
    borderBottomColor: '#eee',
  },
  tab: {
    flex: 1,
    alignItems: 'center',
    paddingVertical: 8,
  },
  activeTab: {
    borderBottomWidth: 2,
    borderBottomColor: '#FF4D4D',
  },
  tabText: {
    fontSize: 16,
    color: '#666',
  },
  activeTabText: {
    color: '#FF4D4D',
    fontWeight: '600',
  },
  ordersList: {
    padding: 16,
  },
  orderCard: {
    backgroundColor: '#f8f8f8',
    padding: 16,
    borderRadius: 8,
    marginBottom: 12,
  },
  orderHeader: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    marginBottom: 8,
  },
  customerName: {
    fontSize: 16,
    fontWeight: '600',
  },
  orderStatus: {
    fontSize: 14,
    fontWeight: '500',
  },
  orderDetails: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    marginBottom: 8,
  },
  itemName: {
    fontSize: 14,
    color: '#666',
  },
  price: {
    fontSize: 14,
    fontWeight: '600',
    color: '#FF4D4D',
  },
  orderTime: {
    fontSize: 12,
    color: '#999',
  },
  actionButtons: {
    flexDirection: 'row',
    marginTop: 12,
  },
  actionButton: {
    flex: 1,
    padding: 8,
    borderRadius: 4,
    alignItems: 'center',
    marginHorizontal: 4,
  },
  acceptButton: {
    backgroundColor: '#4CAF50',
  },
  rejectButton: {
    backgroundColor: '#FF4D4D',
  },
  actionButtonText: {
    color: '#fff',
    fontWeight: '500',
  },
  menuContainer: {
    flex: 1,
    padding: 16,
  },
  addItemButton: {
    flexDirection: 'row',
    backgroundColor: '#FF4D4D',
    padding: 12,
    borderRadius: 8,
    alignItems: 'center',
    justifyContent: 'center',
    marginBottom: 16,
  },
  addItemButtonText: {
    color: '#fff',
    fontWeight: '600',
    marginLeft: 8,
  },
  menuList: {
    paddingBottom: 16,
  },
  menuItem: {
    flexDirection: 'row',
    backgroundColor: '#f8f8f8',
    padding: 12,
    borderRadius: 8,
    marginBottom: 12,
    alignItems: 'center',
  },
  menuItemImage: {
    width: 60,
    height: 60,
    borderRadius: 8,
  },
  menuItemDetails: {
    flex: 1,
    marginLeft: 12,
  },
  menuItemName: {
    fontSize: 16,
    fontWeight: '600',
  },
  menuItemPrice: {
    fontSize: 14,
    color: '#FF4D4D',
    marginTop: 4,
  },
  availabilityButton: {
    paddingHorizontal: 12,
    paddingVertical: 6,
    borderRadius: 4,
  },
  availabilityButtonText: {
    color: '#fff',
    fontSize: 12,
    fontWeight: '500',
  },
});
import React from 'react';
import { View, Text, StyleSheet, TouchableOpacity, ScrollView } from 'react-native';
import { SafeAreaView } from 'react-native-safe-area-context';
import { MaterialIcons } from '@expo/vector-icons';
import { useNavigation } from '@react-navigation/native';
import { toast } from 'sonner-native';

export default function MembershipScreen() {
  const navigation = useNavigation();

  const handleJoin = () => {
    toast.success('Welcome to WCK! You are now a member.');
    navigation.navigate('Home');
  };  return (
    <SafeAreaView style={styles.container}>
      <View style={styles.header}>
        <TouchableOpacity onPress={() => navigation.navigate('Home')}>
          <MaterialIcons name="arrow-back" size={24} color="#333" />
        </TouchableOpacity>
        <Text style={styles.headerTitle}>Become a Seller</Text>
        <View style={{ width: 24 }} />
      </View>

      <ScrollView style={styles.content}>
        <Text style={styles.title}>Start Your Food Business</Text>
        
        <View style={styles.trialBanner}>
          <MaterialIcons name="stars" size={24} color="#FF4D4D" />
          <Text style={styles.trialText}>Try FREE for 9 days</Text>
        </View>
        
        <Text style={styles.priceContainer}>
          <Text style={styles.price}>₹99</Text>
          <Text style={styles.perMonth}>/month</Text>
        </Text>
        <Text style={styles.afterTrial}>after trial period</Text>

        <View style={styles.benefitsContainer}>
          <Text style={styles.benefitsTitle}>Seller Benefits:</Text>
          
          <View style={styles.benefitItem}>
            <MaterialIcons name="store" size={24} color="#4CAF50" />
            <Text style={styles.benefitText}>Set up your virtual restaurant</Text>
          </View>
          
          <View style={styles.benefitItem}>
            <MaterialIcons name="people" size={24} color="#4CAF50" />
            <Text style={styles.benefitText}>Access to thousands of customers</Text>
          </View>
          
          <View style={styles.benefitItem}>
            <MaterialIcons name="payments" size={24} color="#4CAF50" />
            <Text style={styles.benefitText}>No commission fees on sales</Text>
          </View>
          
          <View style={styles.benefitItem}>
            <MaterialIcons name="analytics" size={24} color="#4CAF50" />
            <Text style={styles.benefitText}>Sales analytics and insights</Text>
          </View>
          
          <View style={styles.benefitItem}>
            <MaterialIcons name="support-agent" size={24} color="#4CAF50" />
            <Text style={styles.benefitText}>Priority seller support</Text>
          </View>
        </View>

        <View style={styles.infoBox}>
          <MaterialIcons name="info" size={20} color="#666" />
          <Text style={styles.infoText}>
            Buyers can order for free! Only sellers need a membership.
          </Text>
        </View>

        <TouchableOpacity style={styles.joinButton} onPress={handleJoin}>
          <Text style={styles.joinButtonText}>Start 9-Day Free Trial</Text>
        </TouchableOpacity>

        <Text style={styles.terms}>
          By starting your trial, you agree to our Terms of Service and Privacy Policy. 
          Your card won't be charged during the trial period.
        </Text>
      </ScrollView>
    </SafeAreaView>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff',
  },
  header: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    padding: 16,
    borderBottomWidth: 1,
    borderBottomColor: '#eee',
  },
  headerTitle: {
    fontSize: 18,
    fontWeight: '600',
  },
  content: {
    padding: 20,
  },
  title: {
    fontSize: 28,
    fontWeight: 'bold',
    textAlign: 'center',
    marginBottom: 16,
    color: '#333',
  },
  trialBanner: {
    flexDirection: 'row',
    alignItems: 'center',
    justifyContent: 'center',
    backgroundColor: '#FFF0F0',
    padding: 12,
    borderRadius: 8,
    marginBottom: 16,
  },
  trialText: {
    fontSize: 18,
    fontWeight: '600',
    color: '#FF4D4D',
    marginLeft: 8,
  },
  priceContainer: {
    textAlign: 'center',
    marginBottom: 4,
  },
  price: {
    fontSize: 48,
    fontWeight: 'bold',
    color: '#FF4D4D',
  },
  perMonth: {
    fontSize: 20,
    color: '#666',
  },
  afterTrial: {
    textAlign: 'center',
    color: '#666',
    marginBottom: 24,
  },
  benefitsContainer: {
    backgroundColor: '#f8f8f8',
    padding: 20,
    borderRadius: 12,
    marginBottom: 24,
  },
  benefitsTitle: {
    fontSize: 18,
    fontWeight: '600',
    marginBottom: 16,
  },
  benefitItem: {
    flexDirection: 'row',
    alignItems: 'center',
    marginBottom: 16,
  },
  benefitText: {
    fontSize: 16,
    marginLeft: 12,
    flex: 1,
  },
  infoBox: {
    flexDirection: 'row',
    alignItems: 'center',
    backgroundColor: '#f0f0f0',
    padding: 12,
    borderRadius: 8,
    marginBottom: 24,
  },
  infoText: {
    fontSize: 14,
    color: '#666',
    marginLeft: 8,
    flex: 1,
  },
  joinButton: {
    backgroundColor: '#FF4D4D',
    padding: 16,
    borderRadius: 8,
    alignItems: 'center',
    marginBottom: 16,
  },
  joinButtonText: {
    color: '#fff',
    fontSize: 18,
    fontWeight: '600',
  },
  terms: {
    textAlign: 'center',
    color: '#666',
    fontSize: 12,
    paddingHorizontal: 20,
    marginBottom: 20,
  },
});
