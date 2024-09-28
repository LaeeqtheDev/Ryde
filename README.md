# Ryde - A Ride Sharing App

Ryde is a fully-featured ride-hailing mobile application inspired by Uber. Built using **React Native**, **PostgreSQL**, **TypeScript**, **Stripe**, and **Tailwind CSS**, this app offers seamless, on-demand transportation solutions for both riders and drivers. Whether you need a quick trip across the city or a reliable platform for earning as a driver, Ryde provides an intuitive and efficient experience.

## Features

### Rider Features:
- **Real-time Ride Booking**: Instantly book a ride by entering your pickup and destination locations.
- **Live Driver Tracking**: Track your driver’s live location in real-time after booking a ride.
- **Fare Estimation**: View a price estimate before confirming your ride.
- **Payment Integration**: Secure online payments through **Stripe**, allowing riders to pay via credit card.
- **Ride History**: View details of past rides, including distance traveled, fare breakdown, and trip time.
- **Rating System**: Rate drivers and provide feedback to help improve the service.

### Driver Features:
- **Driver Dashboard**: Manage ride requests, accept or reject trips, and track earnings.
- **Live Navigation**: Get route directions to the rider’s pickup point and destination.
- **Earnings Reports**: View detailed earnings and past ride history.
- **Availability Toggle**: Set your availability status to accept new rides or go offline.

### Admin Panel:
- **User Management**: Add, remove, or manage both riders and drivers.
- **Ride Analytics**: View real-time data and statistics on rides, revenue, and user engagement.
- **Payment Oversight**: Monitor transaction records and ensure successful payments between riders and drivers.

## Tech Stack
- **Frontend**: Built using **React Native** for cross-platform mobile development (iOS & Android).
- **Backend**: Powered by **PostgreSQL** for database management, ensuring scalability and reliability.
- **TypeScript**: Ensures type safety and improves code maintainability across the app.
- **Tailwind CSS**: Used for styling to create a responsive, fast, and customizable UI.
- **Stripe Integration**: Handles secure payments between riders and drivers.

## Getting Started

To get started with the project locally, follow these steps:

1. Clone the repository:
    ```bash
    git clone https://github.com/your-username/ryde-app.git
    cd ryde-app
    ```

2. Install dependencies:
    ```bash
    npm install
    ```

3. Set up the environment variables for **PostgreSQL** and **Stripe**:
    - Create a `.env` file and add your configuration:
      ```bash
      DATABASE_URL=your_postgres_db_url
      STRIPE_SECRET_KEY=your_stripe_key
      ```

4. Start the development server:
    ```bash
    npm start
    ```

5. For iOS:
    ```bash
    npx pod-install
    npm run ios
    ```

6. For Android:
    ```bash
    npm run android
    ```

## Contributions
We welcome contributions to make Ryde even better! Whether it’s fixing bugs, adding new features, or improving documentation, feel free to submit pull requests.

---

