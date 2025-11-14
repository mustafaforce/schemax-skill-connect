# SchemaX - Premier Freelancer Marketplace

SchemaX is a modern, comprehensive freelancer marketplace that connects clients with skilled professionals worldwide. Built with React, TypeScript, and Tailwind CSS, it provides a seamless experience for booking services, managing projects, and growing businesses.

## Features

- **For Clients**: Find and book skilled freelancers instantly
- **For Freelancers**: Create profiles, showcase services, and manage bookings
- **Real-time Chat**: Communicate directly with freelancers
- **Secure Payments**: Protected transactions with instant confirmations
- **Service Management**: Create, edit, and organize your services
- **Availability Scheduling**: Set your availability and manage bookings
- **Review System**: Build trust with a comprehensive rating system

## Tech Stack

- **Frontend**: React 18, TypeScript, Vite
- **UI Framework**: shadcn/ui, Tailwind CSS, Radix UI
- **Backend**: Supabase (PostgreSQL, Authentication, Real-time)
- **State Management**: React Query, Riverpod
- **Routing**: React Router
- **Icons**: Lucide React
- **Forms**: React Hook Form, Zod validation

## Getting Started

### Prerequisites

- Node.js (v16 or higher)
- npm or yarn

### Installation

1. Clone the repository:
```bash
git clone <repository-url>
cd schemax-skill-connect
```

2. Install dependencies:
```bash
npm install
```

3. Start the development server:
```bash
npm run dev
```

4. Open [http://localhost:8080](http://localhost:8080) in your browser.

### Environment Setup

Create a `.env.local` file in the root directory with your Supabase credentials:

```env
VITE_SUPABASE_URL=your_supabase_url
VITE_SUPABASE_ANON_KEY=your_supabase_anon_key
```

## Available Scripts

- `npm run dev` - Start development server
- `npm run build` - Build for production
- `npm run build:dev` - Build for development
- `npm run preview` - Preview production build
- `npm run lint` - Run ESLint

## Project Structure

```
src/
├── components/          # Reusable UI components
│   ├── ui/             # shadcn/ui components
│   ├── home/           # Homepage components
│   └── layout/         # Layout components
├── pages/              # Page components
├── hooks/              # Custom React hooks
├── lib/                # Utilities and configurations
├── integrations/       # External service integrations
└── assets/             # Static assets
```

## Contributing

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Support

For support, email support@schemax.com or join our community discussions.
