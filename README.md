import React, { useState, useEffect } from 'react';
import firebase from 'firebase/app';
import 'firebase/firestore';

// Initialize Firebase
const firebaseConfig = {
  apiKey: 'YOUR_API_KEY',
  authDomain: 'YOUR_AUTH_DOMAIN',
  projectId: 'YOUR_PROJECT_ID',
};

firebase.initializeApp(firebaseConfig);
const db = firebase.firestore();

const App = () => {
  const [items, setItems] = useState([]);
  const [newItem, setNewItem] = useState('');

  useEffect(() => {
    const unsubscribe = db.collection('items').onSnapshot((snapshot) => {
      const updatedItems = snapshot.docs.map((doc) => ({
        id: doc.id,
        ...doc.data(),
      }));
      setItems(updatedItems);
    });

    return () => unsubscribe();
  }, []);

  const handleAddItem = () => {
    db.collection('items')
      .add({ name: newItem })
      .then(() => {
        setNewItem('');
      })
      .catch((error) => {
        console.error('Error adding item: ', error);
      });
  };

  const handleDeleteItem = (itemId) => {
    db.collection('items')
      .doc(itemId)
      .delete()
      .catch((error) => {
        console.error('Error deleting item: ', error);
      });
  };

  return (
    <div>
      <h1>Real-time CRUD App</h1>
      <input
        type="text"
        value={newItem}
        onChange={(e) => setNewItem(e.target.value)}
      />
      <button onClick={handleAddItem}>Add Item</button>
      <ul>
        {items.map((item) => (
          <li key={item.id}>
            {item.name}
            <button onClick={() => handleDeleteItem(item.id)}>Delete</button>
          </li>
        ))}
      </ul>
    </div>
  );
};

export default App;
