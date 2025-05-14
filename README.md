# application-task
import os
from binance.client import Client
import logging
import argparse

# Configure logging
logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(levelname)s - %(message)s')

class BasicBot:
    def _init_(self, api_key, api_secret, testnet=True):
        """
        Initializes the trading bot with API credentials and testnet mode.
        """
        if testnet:
            self.client = Client(api_key, api_secret, tld='com', testnet=True)
            self.base_url = "https://testnet.binancefuture.com"
            logging.info("Using Binance Futures Testnet")
        else:
            self.client = Client(api_key, api_secret, tld='com')
            self.base_url = "https://fapi.binance.com"
            logging.info("Using Binance Futures Live Trading")

        logging.info("Trading Bot initialized")

    def place_order(self, symbol, side, order_type, quantity, price=None, stop_price=None):
        """
        Places an order on Binance Futures Testnet.

        Args:
            symbol (str): Trading pair symbol (e.g., BTCUSDT).
            side (str): Order side ('BUY' or 'SELL').
            order_type (str): Order type ('MARKET' or 'LIMIT').
            quantity (float): Order quantity.
            price (float, optional): Limit price for LIMIT orders. Defaults to None.
            stop_price (float, optional): Stop price for STOP_MARKET/STOP_LIMIT orders. Defaults to None.

        Returns:
            dict: Order details or None if an error occurred.
        """
        try:
            logging.info(f"Placing {order_type} {side} order for {quantity} {symbol}")
            params = {
                'symbol': symbol,
                'side': side,
                'type': order_type,
                'quantity': quantity
            }
            if order_type == 'LIMIT':
                if price is None:
                    logging.error("Limit price is required for LIMIT orders.")
                    return None
                params['timeInForce'] = 'GTC'  # Good Till Cancelled
                params['price'] = price
            elif order_type in ['STOP_MARKET', 'STOP_LIMIT']:
                if stop_price is None:
                    logging.error("Stop price is required for STOP orders.")
                    return None
                params['stopPrice'] = stop_price
                if order_type == 'STOP_LIMIT':
                    if price is None:
                        logging.error("Limit price is required for STOP_LIMIT orders.")
                        return None
                    params['price'] = price
                    params['timeInForce'] = 'GTC'

            response = self.client.futures_create_order(**params)
            logging.info(f"API Request: POST {self.base_url}/fapi/v1/order")
            logging.info(f"API Response: {response}")
            return response
        except Exception as e:
            logging.error(f"Error placing order: {e}")
            return None

    def get_order_status(self, symbol, order_id):
        """
        Retrieves the status of a specific order.

        Args:
            symbol (str): Trading pair symbol.
            order_id (int): The order ID.

        Returns:
            dict: Order status details or None if an error occurred.
        """
        try:
            response = self.client.futures_get_order(symbol=symbol, orderId=order_id)
            logging.info(f"API Request: GET {self.base_url}/fapi/v1/order")
            logging.info(f"API Response: {response}")
            return response
        except Exception as e:
            logging.error(f"Error getting order status: {e}")
            return None

def main():
    parser = argparse.ArgumentParser(description="Simplified Trading Bot for Binance Futures Testnet")
    parser.add_argument("--api_key", required=True, help="Your Binance Testnet API Key")
    parser.add_argument("--api_secret", required=True, help="Your Binance Testnet API Secret")
    args = parser.parse_args()

    api_key = args.api_key
    api_secret = args.api_secret

    bot = BasicBot(api_key, api_secret)

    while True:
        print("\nChoose an action:")
        print("1. Place Market Order")
        print("2. Place Limit Order")
        print("3. Get Order Status")
        print("4. Exit")

        choice = input("Enter your choice: ")

        if choice == '1':
            symbol = input("Enter symbol (e.g., BTCUSDT): ").upper()
            side = input("Enter side (BUY/SELL): ").upper()
            quantity = float(input("Enter quantity: "))
            order_details = bot.place_order(symbol, side, 'MARKET', quantity)
            if order_details:
                print("\nMarket Order Details:")
                print(f"Order ID: {order_details['orderId']}")
                print(f"Symbol: {order_details['symbol']}")
                print(f"Side: {order_details['side']}")
                print(f"Type: {order_details['type']}")
                print(f"Quantity: {order_details['origQty']}")
                print(f"Status: {order_details['status']}")
        elif choice == '2':
            symbol = input("Enter symbol (e.g., BTCUSDT): ").upper()
            side = input("Enter side (BUY/SELL): ").upper()
            quantity = float(input("Enter quantity: "))
            price = float(input("Enter limit price: "))
            order_details = bot.place_order(symbol, side, 'LIMIT', quantity, price=price)
            if order_details:
                print("\nLimit Order Details:")
                print(f"Order ID: {order_details['orderId']}")
                print(f"Symbol: {order_details['symbol']}")
                print(f"Side: {order_details['side']}")
                print(f"Type: {order_details['type']}")
                print(f"Quantity: {order_details['origQty']}")
                print(f"Price: {order_details['price']}")
                print(f"Status: {order_details['status']}")
        elif choice == '3':
            symbol = input("Enter symbol (e.g., BTCUSDT): ").upper()
            order_id = int(input("Enter order ID: "))
            order_status = bot.get_order_status(symbol, order_id)
            if order_status:
                print("\nOrder Status:")
                print(f"Order ID: {order_status['orderId']}")
                print(f"Symbol: {order_status['symbol']}")
                print(f"Side: {order_status['side']}")
                print(f"Type: {order_status['type']}")
                print(f"Quantity: {order_status['origQty']}")
                print(f"Executed Quantity: {order_status['executedQty']}")
                print(f"Status: {order_status['status']}")
        elif choice == '4':
            print("Exiting...")
            break
        else:
            print("Invalid choice. Please try again.")

if _name_ == "_main_":
    main()

How to Use:
 * Register and Activate Binance Testnet Account:
   * Go to Binance Futures Testnet.
   * Register for a new account or log in if you already have one.
   * Follow the instructions to activate your Futures Testnet account.
 * Generate API Credentials:
   * Log in to your Binance Futures Testnet account.
   * Navigate to the API Management section (usually under your profile).
   * Create a new API key and secret. Make sure to enable Futures trading for this API key.
   * Important: Keep your API key and secret safe. Do not share them.
 * Save the Code:
   * Save the Python code above as a .py file (e.g., trading_bot.py).
 * Install python-binance Library:
   * If you don't have it installed, open your terminal or command prompt and run:
     pip install python-binance

 * Run the Bot:
   * Open your terminal or command prompt.
   * Navigate to the directory where you saved the trading_bot.py file.
   * Run the script with your API key and secret as command-line arguments:
     python trading_bot.py --api_key YOUR_TESTNET_API_KEY --api_secret YOUR_TESTNET_API_SECRET

     Replace YOUR_TESTNET_API_KEY and YOUR_TESTNET_API_SECRET with your actual testnet API key and secret.
 * Interact with the Bot:
   * The script will present a menu in the terminal.
   * Choose an option (1, 2, 3, or 4).
   * Follow the prompts to enter the required information for placing orders or checking status.
Explanation of the Code:
 * Import Libraries: Imports necessary libraries (os, binance.client, logging, argparse).
 * Logging Configuration: Sets up basic logging to the console.
 * BasicBot Class:
   * _init_: Initializes the Binance client in testnet mode using the provided API key and secret. Sets the base_url for API calls.
   * place_order:
     * Takes parameters for symbol, side (BUY/SELL), order type (MARKET/LIMIT), quantity, and optional price (for LIMIT orders) and stop price (for STOP orders).
     * Constructs the parameters dictionary for the Binance API.
     * Uses self.client.futures_create_order() to place the order.
     * Logs the API request and response.
     * Includes basic error handling using a try-except block.
   * get_order_status:
     * Takes symbol and order ID as input.
     * Uses self.client.futures_get_order() to retrieve the order status.
     * Logs the API request and response.
     * Includes basic error handling.
 * main Function:
   * Uses argparse to securely get the API key and secret from the command line. This is a better practice than hardcoding them.
   * Creates an instance of the BasicBot class.
   * Presents a command-line interface (CLI) loop:
     * Allows the user to choose between placing a market order, placing a limit order, getting order status, or exiting.
     * Prompts the user for the necessary details for each action.
     * Calls the corresponding methods of the BasicBot class.
     * Prints the order details or status to the console.
     * Handles invalid choices.
 * if _name_ == "_main_":: Ensures that the main function is executed only when the script is run directly.
Optional Advanced Order Types (Example - Stop-Limit):
You can extend the place_order function to include support for STOP_LIMIT orders. Here's how you could modify the main function and the place_order method:
In main function, add a new choice:
        print("1. Place Market Order")
        print("2. Place Limit Order")
        print("3. Place Stop-Limit Order")  # Added option
        print("4. Get Order Status")
        print("5. Exit")

        elif choice == '3':
            symbol = input("Enter symbol (e.g., BTCUSDT): ").upper()
            side = input("Enter side (BUY/SELL): ").upper()
            quantity = float(input("Enter quantity: "))
            price = float(input("Enter limit price: "))
            stop_price = float(input("Enter stop price: "))
            order_details = bot.place_order(symbol, side, 'STOP_LIMIT', quantity, price=price, stop_price=stop_price)
            if order_details:
                print("\nStop-Limit Order Details:")
                print(f"Order ID: {order_details['orderId']}")
                # ... (print other details)
                print(f"Stop Price: {order_details['stopPrice']}")
                print(f"Price: {order_details['price']}")
                print(f"Status: {order_details['status']}")

In BasicBot.place_order method, add the elif block:
            elif order_type in ['STOP_MARKET', 'STOP_LIMIT']:
                # ... (existing code for STOP_MARKET) ...
                if order_type == 'STOP_LIMIT':
                    if price is None:
                        logging.error("Limit price is required for STOP_LIMIT orders.")
                        return None
                    params['price'] = price
                    params['timeInForce'] = 'GTC'

Remember to explore the python-binance documentation for details on other advanced order types like OCO.
This code provides a solid foundation for your simplified trading bot. Good luck with the hiring process!
