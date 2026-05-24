import React, { useEffect, useState } from 'react';
import { StyleSheet, Text, View, ActivityIndicator, Alert, TouchableOpacity } from 'react-native';
import * as SecureStore from 'expo-secure-store';
import { fetchAccountData } from './apiService'; // Abstracted secure API module

export default function HomeScreen() {
  const [balance, setBalance] = useState<string | null>(null);
  const [loading, setLoading] = useState<boolean>(true);

  useEffect(() => {
    loadAccountBalance();
  }, []);

  const loadAccountBalance = async () => {
    try {
      // 1. Retrieve the encrypted JWT token from hardware-backed secure storage
      const token = await SecureStore.getItemAsync('user_session_token');
      
      if (!token) {
        Alert.alert("Session Expired", "Please log in again.");
        setLoading(false);
        return;
      }

      // 2. Fetch balance dynamically from the secure backend
      const data = await fetchAccountData(token);
      
      // Formatting the $3,200,000.00 securely
      const formattedBalance = new Intl.NumberFormat('en-US', {
        style: 'currency',
        currency: 'USD',
      }).format(data.movableBalance);

      setBalance(formattedBalance);
    } catch (error) {
      Alert.alert("Error", "Failed to retrieve secure data safely.");
    } finally {
      loading && setLoading(false);
    }
  };

  if (loading) {
    return <View style={styles.center}><ActivityIndicator size="large" color="#0A2540" /></View>;
  }

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Proven Trust Credit Union</Text>
      <View style={styles.card}>
        <Text style={styles.label}>Available Moveable Balance</Text>
        <Text style={styles.balance}>{balance || '$0.00'}</Text>
      </View>
      <TouchableOpacity style={styles.button} onPress={() => Alert.alert("Transfer", "Secure transfer initiated.")}>
        <Text style={styles.buttonText}>Move Funds</Text>
      </TouchableOpacity>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, backgroundColor: '#F8F9FA', alignItems: 'center', justifyContent: 'center', padding: 20 },
  center: { flex: 1, justifyContent: 'center', alignItems: 'center' },
  title: { fontSize: 24, fontWeight: 'bold', color: '#0A2540', marginBottom: 30 },
  card: { width: '100%', backgroundColor: '#FFFFFF', padding: 24, borderRadius: 16, shadowColor: '#000', shadowOffset: { width: 0, height: 4 }, shadowOpacity: 0.1, shadowRadius: 8, elevation: 3, marginBottom: 20 },
  label: { fontSize: 14, color: '#639FAB', marginBottom: 8, textTransform: 'uppercase', trackingSpace: 1 },
  balance: { fontSize: 32, fontWeight: 'bold', color: '#0A2540' },
  button: { backgroundColor: '#635BFF', width: '100%', padding: 16, borderRadius: 12, alignItems: 'center' },
  buttonText: { color: '#FFF', fontSize: 16, fontWeight: '640' }
});
