# GTTC Groningen - Website

Official website for **Groninger Tafeltennis Club (GTTC)**, a table tennis club based in Groningen, Netherlands.

This is a modern, fully-featured website built with **React**, **TypeScript**, **Vite**, and **Tailwind CSS**, featuring a dynamic admin dashboard for content management via Supabase.

## 🎯 About GTTC

GTTC is a vibrant table tennis club in Groningen that welcomes members of all ages and skill levels. We offer:
- **Training programs** for beginners and advanced players
- **Competition opportunities** at various levels
- **Social events and tournaments** throughout the year
- **Community engagement** with local table tennis enthusiasts

Whether you're a seasoned player or just looking to discover the sport, GTTC is the place for you!

## 🌐 Website Features

- **Home Page** - Hero banner, club information, news, training & competition sections
- **About Page** - History and information about GTTC
- **News Section** - Latest updates and announcements
- **Training** - Training schedule and programs
- **Competition** - Competition details and standings
- **Membership** - Membership options and benefits
- **Agenda** - Event calendar and schedule
- **Contact** - Get in touch with the club
- **Vacancies** - Current job and volunteer opportunities
- **Admin Dashboard** - Content management system for editing homepage, navigation, and site content (password-protected)
- **Multilingual Support** - Dutch language interface with i18n framework ready for expansion

## 🛠️ Tech Stack

- **Frontend Framework**: React 19
- **Language**: TypeScript
- **Build Tool**: Vite
- **Styling**: Tailwind CSS
- **Routing**: React Router v7
- **Backend/CMS**: Supabase
- **Linting**: ESLint
- **State Management**: React Hooks
- **Icons**: Lucide React, Font Awesome, Remix Icon
- **Internationalization**: i18next
- **Form Validation**: hCaptcha
- **Analytics**: Recharts

## 🚀 Getting Started

### Prerequisites
- Node.js (v18 or higher)
- npm or yarn

### Installation

```bash
# Clone the repository
git clone https://github.com/hosammr/GTTC.git
cd GTTC

# Install dependencies
npm install

# Set up environment variables
cp .env.example .env.local
# Edit .env.local with your Supabase credentials
```

### Development

```bash
# Start the development server
npm run dev

# Open http://localhost:3000 in your browser
```

### Building for Production

```bash
# Build the project
npm run build

# Preview the production build
npm run preview
```

### Linting

```bash
# Check for linting errors
npm run lint

# Type checking
npm run type-check
```

## 📋 Project Structure

```
src/
├── components/        # Reusable React components
├── pages/             # Page components
├── router/            # Route configuration
├── hooks/             # Custom React hooks
├── context/           # React context for state management
├── styles/            # Global styles
├── utils/             # Utility functions
├── assets/            # Images and static assets
└── main.tsx           # Application entry point

public/
├── vite.svg           # Favicon
└── [other assets]

Configuration files:
├── vite.config.ts     # Vite configuration
├── tailwind.config.ts # Tailwind CSS configuration
├── tsconfig.json      # TypeScript configuration
├── eslint.config.ts   # ESLint configuration
└── postcss.config.ts  # PostCSS configuration
```

## 🔐 Environment Variables

This project requires the following environment variables (see `.env.example`):

```
VITE_PUBLIC_SUPABASE_URL=your_supabase_url
VITE_PUBLIC_SUPABASE_ANON_KEY=your_supabase_anon_key
```

⚠️ **Never commit `.env` files to the repository!** Use GitHub Secrets for sensitive data in CI/CD.

## 🔄 GitHub Pages Deployment

This project is automatically deployed to GitHub Pages on every push to the `main` branch using GitHub Actions.

**Live Site**: [hosammr.github.io/GTTC](https://hosammr.github.io/GTTC)

### Deployment Configuration

- **Build Output**: `out/` directory
- **Base Path**: `/GTTC`
- **Automatic Builds**: GitHub Actions workflow on every push
- **Environment Variables**: Set via GitHub Secrets for production builds

### Setting Up GitHub Secrets

To enable production builds with Supabase integration:

1. Go to **Settings** → **Secrets and variables** → **Actions**
2. Add the following secrets:
   - `VITE_PUBLIC_SUPABASE_URL`
   - `VITE_PUBLIC_SUPABASE_ANON_KEY`

## 📝 Contributing

We welcome contributions! Please feel free to submit a Pull Request with improvements, bug fixes, or new features.

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 📞 Contact

For questions about the website or GTTC:
- **Email**: [Add contact email]
- **Phone**: [Add phone number]
- **Address**: Groningen, Netherlands
- **Social Media**: [Add social media links]

## 🙏 Acknowledgments

Built with ❤️ for the Groninger Tafeltennis Club community.

---

**Last Updated**: June 2026