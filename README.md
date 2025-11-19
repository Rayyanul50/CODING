
from tkinter import *
import sys
import warnings
import json
import tkinter as tk
from tkinter import messagebox, ttk
from PIL import Image, ImageTk, ImageDraw, ImageFont
import webbrowser
import os
import hashlib
import random

warnings.filterwarnings("ignore")

class MovieStreamingApp:
    def __init__(self, root):
        self.root = root
        
        # Initialize attributes FIRST
        self.data_file = "users.json"
        self.images_dir = "movie_images"
        self.users = self.load_users()
        self.current_user = None
        
        # Color scheme
        self.colors = {
            'bg': '#0a0a0a',
            'card_bg': '#1a1a1a',
            'accent': '#e50914',
            'text': '#ffffff',
            'text_secondary': '#b3b3b3',
        }
        
        # TEST: Check PIL first
        self.test_pil()
        
        self.setup_images_directory()
        
        # Create UI LAST
        self.create_main_page()
        
    def create_main_page(self):
        # Clear any existing widgets
        for widget in self.root.winfo_children():
            widget.destroy()
            
        self.root.title('FROZZLE - Premium Streaming Platform')
        self.root.config(bg='#0a0a0a')
        self.center_window(1000, 700)
        
        # Main container
        main_frame = tk.Frame(self.root, bg='#0a0a0a')
        main_frame.pack(fill=tk.BOTH, expand=True, padx=50, pady=50)
        
        # Welcome message
        welcome_label = tk.Label(main_frame, 
                        text="Welcome to FROZZLE",
                        font=('Arial', 32, 'bold'),
                        fg='white',
                        bg='#0a0a0a')
        welcome_label.pack(pady=20)
        
        # Subtitle
        subtitle_label = tk.Label(main_frame,
                         text="Premium Streaming Platform",
                         font=('Arial', 18),
                         fg='#b3b3b3',
                         bg='#0a0a0a')
        subtitle_label.pack(pady=10)
        
        # Buttons frame
        button_frame = tk.Frame(main_frame, bg='#0a0a0a')
        button_frame.pack(pady=50)
        
        # Login button
        login_btn = tk.Button(button_frame,
                            text="Login",
                            font=('Arial', 16),
                            bg='#e50914',
                            fg='white',
                            width=15,
                            height=2,
                            command=self.open_login)
        login_btn.pack(side=tk.LEFT, padx=20)
        
        # Register button
        register_btn = tk.Button(button_frame,
                               text="Register",
                               font=('Arial', 16),
                               bg='#333333',
                               fg='white',
                               width=15,
                               height=2,
                               command=self.open_register)
        register_btn.pack(side=tk.LEFT, padx=20)
        
        # Status label
        self.status_label = tk.Label(main_frame,
                                   text="",
                                   font=('Arial', 12),
                                   fg='#b3b3b3',
                                   bg='#0a0a0a')
        self.status_label.pack(pady=20)
        
        # Update status
        self.update_status()
    
    def open_login(self):
        """Open login form"""
        # Clear existing widgets
        for widget in self.root.winfo_children():
            widget.destroy()
            
        self.root.title('FROZZLE - Login')
        self.root.config(bg='#0a0a0a')
        self.center_window(400, 500)
        
        # Main container
        main_frame = tk.Frame(self.root, bg='#0a0a0a')
        main_frame.pack(fill=tk.BOTH, expand=True, padx=40, pady=40)
        
        # Title
        title_label = tk.Label(main_frame,
                             text="Login to FROZZLE",
                             font=('Arial', 24, 'bold'),
                             fg='white',
                             bg='#0a0a0a')
        title_label.pack(pady=20)
        
        # Username
        tk.Label(main_frame, text="Username:", fg='white', bg='#0a0a0a', font=('Arial', 12)).pack(anchor='w', pady=(20,5))
        username_entry = tk.Entry(main_frame, font=('Arial', 14), width=25)
        username_entry.pack(fill='x', pady=5)
        
        # Password
        tk.Label(main_frame, text="Password:", fg='white', bg='#0a0a0a', font=('Arial', 12)).pack(anchor='w', pady=(20,5))
        password_entry = tk.Entry(main_frame, font=('Arial', 14), width=25, show='*')
        password_entry.pack(fill='x', pady=5)
        
        # Login button
        login_btn = tk.Button(main_frame,
                            text="Login",
                            font=('Arial', 14),
                            bg='#e50914',
                            fg='white',
                            width=20,
                            height=2,
                            command=lambda: self.login_user(username_entry.get(), password_entry.get()))
        login_btn.pack(pady=30)
        
        # Back button
        back_btn = tk.Button(main_frame,
                           text="← Back",
                           font=('Arial', 12),
                           bg='#333333',
                           fg='white',
                           command=self.create_main_page)
        back_btn.pack(pady=10)
    
    def open_register(self):
        """Open registration form"""
        # Clear existing widgets
        for widget in self.root.winfo_children():
            widget.destroy()
            
        self.root.title('FROZZLE - Register')
        self.root.config(bg='#0a0a0a')
        self.center_window(400, 600)
        
        # Main container
        main_frame = tk.Frame(self.root, bg='#0a0a0a')
        main_frame.pack(fill=tk.BOTH, expand=True, padx=40, pady=40)
        
        # Title
        title_label = tk.Label(main_frame,
                             text="Join FROZZLE",
                             font=('Arial', 24, 'bold'),
                             fg='white',
                             bg='#0a0a0a')
        title_label.pack(pady=20)
        
        # Username
        tk.Label(main_frame, text="Username:", fg='white', bg='#0a0a0a', font=('Arial', 12)).pack(anchor='w', pady=(20,5))
        username_entry = tk.Entry(main_frame, font=('Arial', 14), width=25)
        username_entry.pack(fill='x', pady=5)
        
        # Email
        tk.Label(main_frame, text="Email:", fg='white', bg='#0a0a0a', font=('Arial', 12)).pack(anchor='w', pady=(20,5))
        email_entry = tk.Entry(main_frame, font=('Arial', 14), width=25)
        email_entry.pack(fill='x', pady=5)
        
        # Password
        tk.Label(main_frame, text="Password:", fg='white', bg='#0a0a0a', font=('Arial', 12)).pack(anchor='w', pady=(20,5))
        password_entry = tk.Entry(main_frame, font=('Arial', 14), width=25, show='*')
        password_entry.pack(fill='x', pady=5)
        
        # Confirm Password
        tk.Label(main_frame, text="Confirm Password:", fg='white', bg='#0a0a0a', font=('Arial', 12)).pack(anchor='w', pady=(20,5))
        confirm_password_entry = tk.Entry(main_frame, font=('Arial', 14), width=25, show='*')
        confirm_password_entry.pack(fill='x', pady=5)
        
        # Register button
        register_btn = tk.Button(main_frame,
                               text="Create Account",
                               font=('Arial', 14),
                               bg='#e50914',
                               fg='white',
                               width=20,
                               height=2,
                               command=lambda: self.register_user(
                                   username_entry.get(),
                                   email_entry.get(),
                                   password_entry.get(),
                                   confirm_password_entry.get()
                               ))
        register_btn.pack(pady=30)
        
        # Back button
        back_btn = tk.Button(main_frame,
                           text="← Back",
                           font=('Arial', 12),
                           bg='#333333',
                           fg='white',
                           command=self.create_main_page)
        back_btn.pack(pady=10)
    
    def login_user(self, username, password):
        """Login user"""
        if not username or not password:
            messagebox.showerror("Error", "Please fill in all fields")
            return
            
        if username in self.users:
            hashed_password = self.hash_password(password)
            if self.users[username]['password'] == hashed_password:
                self.current_user = username
                messagebox.showinfo("Success", f"Welcome back, {username}!")
                self.create_home_page()
            else:
                messagebox.showerror("Error", "Invalid password")
        else:
            messagebox.showerror("Error", "Username not found")
    
    def register_user(self, username, email, password, confirm_password):
        """Register new user"""
        if not all([username, email, password, confirm_password]):
            messagebox.showerror("Error", "Please fill in all fields")
            return
            
        # Validate username
        valid_username, msg = self.validate_username(username)
        if not valid_username:
            messagebox.showerror("Error", msg)
            return
            
        # Validate password
        valid_password, msg = self.validate_password(password)
        if not valid_password:
            messagebox.showerror("Error", msg)
            return
            
        if password != confirm_password:
            messagebox.showerror("Error", "Passwords do not match")
            return
            
        if username in self.users:
            messagebox.showerror("Error", "Username already exists")
            return
            
        # Register user
        self.users[username] = {
            'email': email,
            'password': self.hash_password(password),
            'plan': 'basic'
        }
        self.save_users()
        messagebox.showinfo("Success", "Account created successfully!")
        self.create_main_page()
    
    def create_home_page(self):
        """Create main home page after login"""
        # Clear existing widgets
        for widget in self.root.winfo_children():
            widget.destroy()
            
        self.root.title(f'FROZZLE - Welcome {self.current_user}')
        self.root.config(bg='#0a0a0a')
        self.center_window(1200, 800)
        
        # Header
        header = tk.Frame(self.root, bg='#1a1a1a', height=60)
        header.pack(fill='x', padx=0, pady=0)
        header.pack_propagate(False)
        
        # Logo
        logo_label = tk.Label(header, text="FROZZLE", font=('Arial', 24, 'bold'), 
                             fg='#e50914', bg='#1a1a1a')
        logo_label.pack(side='left', padx=20, pady=10)
        
        # Navigation
        nav_frame = tk.Frame(header, bg='#1a1a1a')
        nav_frame.pack(side='left', padx=50)
        
        home_btn = tk.Button(nav_frame, text="Home", font=('Arial', 12), 
                           bg='#1a1a1a', fg='white', bd=0,
                           command=self.create_home_page)
        home_btn.pack(side='left', padx=10)
        
        movies_btn = tk.Button(nav_frame, text="Movies", font=('Arial', 12), 
                             bg='#1a1a1a', fg='white', bd=0,
                             command=self.open_movies)
        movies_btn.pack(side='left', padx=10)
        
        series_btn = tk.Button(nav_frame, text="TV Series", font=('Arial', 12), 
                             bg='#1a1a1a', fg='white', bd=0,
                             command=self.open_webseries)
        series_btn.pack(side='left', padx=10)
        
        # User info
        user_frame = tk.Frame(header, bg='#1a1a1a')
        user_frame.pack(side='right', padx=20)
        
        user_label = tk.Label(user_frame, text=f"Welcome, {self.current_user}", 
                            font=('Arial', 12), fg='white', bg='#1a1a1a')
        user_label.pack(side='left', padx=10)
        
        logout_btn = tk.Button(user_frame, text="Logout", font=('Arial', 10), 
                             bg='#e50914', fg='white',
                             command=self.logout)
        logout_btn.pack(side='left', padx=5)
        
        # Main content
        content_frame = tk.Frame(self.root, bg='#0a0a0a')
        content_frame.pack(fill='both', expand=True, padx=20, pady=20)
        
        # Welcome section
        welcome_section = tk.Frame(content_frame, bg='#0a0a0a')
        welcome_section.pack(fill='x', pady=20)
        
        welcome_label = tk.Label(welcome_section, 
                               text=f"Hello, {self.current_user}!",
                               font=('Arial', 28, 'bold'),
                               fg='white',
                               bg='#0a0a0a')
        welcome_label.pack(anchor='w')
        
        subtitle_label = tk.Label(welcome_section,
                                text="What would you like to watch today?",
                                font=('Arial', 16),
                                fg='#b3b3b3',
                                bg='#0a0a0a')
        subtitle_label.pack(anchor='w', pady=(5, 20))
        
        # Quick access buttons
        quick_frame = tk.Frame(content_frame, bg='#0a0a0a')
        quick_frame.pack(fill='x', pady=20)
        
        movies_quick_btn = tk.Button(quick_frame,
                                   text="🎬 Browse Movies",
                                   font=('Arial', 14),
                                   bg='#e50914',
                                   fg='white',
                                   width=20,
                                   height=2,
                                   command=self.open_movies)
        movies_quick_btn.pack(side='left', padx=10)
        
        series_quick_btn = tk.Button(quick_frame,
                                   text="📺 Browse Series",
                                   font=('Arial', 14),
                                   bg='#333333',
                                   fg='white',
                                   width=20,
                                   height=2,
                                   command=self.open_webseries)
        series_quick_btn.pack(side='left', padx=10)
        
        # Featured content
        featured_frame = tk.Frame(content_frame, bg='#0a0a0a')
        featured_frame.pack(fill='x', pady=20)
        
        featured_label = tk.Label(featured_frame,
                                text="Featured Content",
                                font=('Arial', 20, 'bold'),
                                fg='white',
                                bg='#0a0a0a')
        featured_label.pack(anchor='w')
        
        # Featured movies grid
        featured_movies = self.get_movies_data()[:6]  # First 6 movies
        self.create_content_grid(featured_frame, featured_movies, "Movies")
        
        # Featured series
        featured_series = self.get_series_data()[:6]  # First 6 series
        self.create_content_grid(featured_frame, featured_series, "TV Series")
    
    def create_content_grid(self, parent, content, title):
        """Create a grid of content posters"""
        section_frame = tk.Frame(parent, bg='#0a0a0a')
        section_frame.pack(fill='x', pady=20)
        
        # Section title
        title_label = tk.Label(section_frame,
                             text=title,
                             font=('Arial', 18, 'bold'),
                             fg='white',
                             bg='#0a0a0a')
        title_label.pack(anchor='w', pady=(0, 10))
        
        # Content grid
        grid_frame = tk.Frame(section_frame, bg='#0a0a0a')
        grid_frame.pack(fill='x')
        
        for i, item in enumerate(content):
            if i >= 6:  # Limit to 6 items per row
                break
                
            item_frame = tk.Frame(grid_frame, bg='#1a1a1a', width=180, height=280)
            item_frame.pack(side='left', padx=10, pady=5)
            item_frame.pack_propagate(False)
            
            # Load and display image
            image_path = self.get_image_path(item['image'])
            if image_path and os.path.exists(image_path):
                try:
                    img = Image.open(image_path)
                    img = img.resize((160, 200), Image.Resampling.LANCZOS)
                    photo = ImageTk.PhotoImage(img)
                    
                    img_label = tk.Label(item_frame, image=photo, bg='#1a1a1a')
                    img_label.image = photo  # Keep reference
                    img_label.pack(pady=10)
                except Exception as e:
                    # Fallback if image loading fails
                    fallback_label = tk.Label(item_frame, text="No Image", 
                                            bg='#1a1a1a', fg='white', width=20, height=10)
                    fallback_label.pack(pady=10)
            else:
                fallback_label = tk.Label(item_frame, text="No Image", 
                                        bg='#1a1a1a', fg='white', width=20, height=10)
                fallback_label.pack(pady=10)
            
            # Title
            title_text = item['name'] if len(item['name']) <= 20 else item['name'][:17] + "..."
            title_label = tk.Label(item_frame, text=title_text,
                                 font=('Arial', 10, 'bold'),
                                 fg='white', bg='#1a1a1a',
                                 wraplength=160)
            title_label.pack(pady=5)
            
            # Genre
            genre_label = tk.Label(item_frame, text=item['genre'],
                                 font=('Arial', 8),
                                 fg='#b3b3b3', bg='#1a1a1a')
            genre_label.pack(pady=2)
            
            # Watch button
            watch_btn = tk.Button(item_frame, text="Watch Now",
                                font=('Arial', 9),
                                bg='#e50914', fg='white',
                                command=lambda url=item['url']: self.open_content_window(url))
            watch_btn.pack(pady=5)
    
    def open_movies(self):
        """Open movies browsing page"""
        self.create_browse_page("Movies", self.get_movies_data())
    
    def open_webseries(self):
        """Open TV series browsing page"""
        self.create_browse_page("TV Series", self.get_series_data())
    
    def create_browse_page(self, title, content):
        """Create a browsing page for movies or series"""
        # Clear existing widgets
        for widget in self.root.winfo_children():
            widget.destroy()
            
        self.root.title(f'FROZZLE - {title}')
        self.root.config(bg='#0a0a0a')
        self.center_window(1200, 800)
        
        # Header (same as home page)
        header = tk.Frame(self.root, bg='#1a1a1a', height=60)
        header.pack(fill='x', padx=0, pady=0)
        header.pack_propagate(False)
        
        logo_label = tk.Label(header, text="FROZZLE", font=('Arial', 24, 'bold'), 
                             fg='#e50914', bg='#1a1a1a')
        logo_label.pack(side='left', padx=20, pady=10)
        
        nav_frame = tk.Frame(header, bg='#1a1a1a')
        nav_frame.pack(side='left', padx=50)
        
        tk.Button(nav_frame, text="Home", font=('Arial', 12), 
                bg='#1a1a1a', fg='white', bd=0, command=self.create_home_page).pack(side='left', padx=10)
        tk.Button(nav_frame, text="Movies", font=('Arial', 12), 
                bg='#1a1a1a', fg='white', bd=0, command=self.open_movies).pack(side='left', padx=10)
        tk.Button(nav_frame, text="TV Series", font=('Arial', 12), 
                bg='#1a1a1a', fg='white', bd=0, command=self.open_webseries).pack(side='left', padx=10)
        
        user_frame = tk.Frame(header, bg='#1a1a1a')
        user_frame.pack(side='right', padx=20)
        tk.Label(user_frame, text=f"Welcome, {self.current_user}", 
                font=('Arial', 12), fg='white', bg='#1a1a1a').pack(side='left', padx=10)
        tk.Button(user_frame, text="Logout", font=('Arial', 10), 
                bg='#e50914', fg='white', command=self.logout).pack(side='left', padx=5)
        
        # Main content
        content_frame = tk.Frame(self.root, bg='#0a0a0a')
        content_frame.pack(fill='both', expand=True, padx=20, pady=20)
        
        # Page title
        title_label = tk.Label(content_frame, text=title,
                             font=('Arial', 28, 'bold'),
                             fg='white', bg='#0a0a0a')
        title_label.pack(anchor='w', pady=20)
        
        # Content grid
        self.create_poster_grid(content_frame, content)
    
    def create_poster_grid(self, parent, content):
        """Create a scrollable grid of posters"""
        # Create canvas and scrollbar
        canvas = tk.Canvas(parent, bg='#0a0a0a', highlightthickness=0)
        scrollbar = tk.Scrollbar(parent, orient='vertical', command=canvas.yview)
        scrollable_frame = tk.Frame(canvas, bg='#0a0a0a')
        
        scrollable_frame.bind(
            "<Configure>",
            lambda e: canvas.configure(scrollregion=canvas.bbox("all"))
        )
        
        canvas.create_window((0, 0), window=scrollable_frame, anchor="nw")
        canvas.configure(yscrollcommand=scrollbar.set)
        
        # Pack canvas and scrollbar
        canvas.pack(side="left", fill="both", expand=True)
        scrollbar.pack(side="right", fill="y")
        
        # Create grid of content
        row_frame = None
        for i, item in enumerate(content):
            if i % 4 == 0:  # 4 items per row
                row_frame = tk.Frame(scrollable_frame, bg='#0a0a0a')
                row_frame.pack(fill='x', pady=10)
            
            self.create_poster_item(row_frame, item)
    
    def create_poster_item(self, parent, item):
        """Create individual poster item"""
        item_frame = tk.Frame(parent, bg='#1a1a1a', width=220, height=320)
        item_frame.pack(side='left', padx=15, pady=10)
        item_frame.pack_propagate(False)
        
        # Load and display image
        image_path = self.get_image_path(item['image'])
        if image_path and os.path.exists(image_path):
            try:
                img = Image.open(image_path)
                img = img.resize((200, 250), Image.Resampling.LANCZOS)
                photo = ImageTk.PhotoImage(img)
                
                img_label = tk.Label(item_frame, image=photo, bg='#1a1a1a', cursor='hand2')
                img_label.image = photo
                img_label.pack(pady=10)
                img_label.bind("<Button-1>", lambda e, url=item['url']: self.open_content_window(url))
            except:
                self.create_fallback_poster(item_frame, item['name'])
        else:
            self.create_fallback_poster(item_frame, item['name'])
        
        # Title
        title_text = item['name'] if len(item['name']) <= 25 else item['name'][:22] + "..."
        title_label = tk.Label(item_frame, text=title_text,
                             font=('Arial', 11, 'bold'),
                             fg='white', bg='#1a1a1a',
                             wraplength=180, cursor='hand2')
        title_label.pack(pady=5)
        title_label.bind("<Button-1>", lambda e, url=item['url']: self.open_content_window(url))
        
        # Genre
        genre_label = tk.Label(item_frame, text=item['genre'],
                             font=('Arial', 9),
                             fg='#b3b3b3', bg='#1a1a1a')
        genre_label.pack(pady=2)
        
        # Watch button
        watch_btn = tk.Button(item_frame, text="▶ Watch Now",
                            font=('Arial', 10),
                            bg='#e50914', fg='white',
                            command=lambda url=item['url']: self.open_content_window(url))
        watch_btn.pack(pady=8)
    
    def create_fallback_poster(self, parent, title):
        """Create fallback poster when image fails to load"""
        fallback_frame = tk.Frame(parent, bg='#2a2a2a', width=200, height=250)
        fallback_frame.pack(pady=10)
        fallback_frame.pack_propagate(False)
        
        title_label = tk.Label(fallback_frame, text=title,
                             font=('Arial', 12, 'bold'),
                             fg='white', bg='#2a2a2a',
                             wraplength=180)
        title_label.pack(expand=True)
    
    def open_content_window(self, url):
        """Open content in web browser"""
        try:
            webbrowser.open(url)
            messagebox.showinfo("Success", "Opening in your web browser...")
        except Exception as e:
            messagebox.showerror("Error", f"Could not open content: {str(e)}")
    
    def logout(self):
        """Logout user"""
        self.current_user = None
        self.create_main_page()
    
    def update_status(self):
        """Update status with image count"""
        if os.path.exists(self.images_dir):
            files = [f for f in os.listdir(self.images_dir) if f.endswith(('.jpg', '.png', '.jpeg'))]
            self.status_label.config(text=f"Ready with {len(files)} movies & series available")
        else:
            self.status_label.config(text="Setting up...")

    # ... [REST OF YOUR METHODS REMAIN THE SAME - test_pil, center_window, setup_images_directory, etc.] ...

    def test_pil(self):
        """Test if PIL is working"""
        try:
            test_img = Image.new('RGB', (10, 10), color='red')
            test_img.save('test_delete_me.jpg')
            if os.path.exists('test_delete_me.jpg'):
                os.remove('test_delete_me.jpg')
            print("✓ PIL is working correctly")
            return True
        except Exception as e:
            print(f"✗ PIL Error: {e}")
            messagebox.showerror("PIL Error", f"Pillow library not working: {e}\n\nRun: pip install pillow")
            return False

    def center_window(self, width, height):
        screen_width = self.root.winfo_screenwidth()
        screen_height = self.root.winfo_screenheight()
        x = (screen_width - width) // 2
        y = (screen_height - height) // 2
        self.root.geometry(f"{width}x{height}+{x}+{y}")

    def setup_images_directory(self):
        """Create images directory and generate pictures"""
        print("Setting up images directory...")
        
        if not os.path.exists(self.images_dir):
            print(f"Creating directory: {self.images_dir}")
            os.makedirs(self.images_dir)
        
        existing_files = [f for f in os.listdir(self.images_dir) if f.endswith(('.jpg', '.png', '.jpeg'))]
        print(f"Found {len(existing_files)} existing images")
        
        if len(existing_files) < 50:
            print("Generating missing images...")
            self.generate_all_images()
        else:
            print("Images already exist, skipping generation")

    def generate_all_images(self):
        """Generate all movie and series posters"""
        try:
            print("Starting image generation...")
            
            movies = self.get_movies_data()
            print(f"Generating {len(movies)} movie posters...")
            
            for i, movie in enumerate(movies):
                image_path = os.path.join(self.images_dir, f"{movie['image']}.jpg")
                if not os.path.exists(image_path):
                    self.create_simple_poster(movie["name"], movie["image"], movie["genre"])
                if (i + 1) % 20 == 0:
                    print(f"Processed {i + 1}/{len(movies)} movies")
            
            series = self.get_series_data()
            print(f"Generating {len(series)} series posters...")
            
            for i, show in enumerate(series):
                image_path = os.path.join(self.images_dir, f"{show['image']}.jpg")
                if not os.path.exists(image_path):
                    self.create_simple_poster(show["name"], show["image"], show["genre"], is_series=True)
                if (i + 1) % 10 == 0:
                    print(f"Processed {i + 1}/{len(series)} series")
            
            print("✓ All images generated successfully!")
            
        except Exception as e:
            print(f"Error generating images: {e}")
            self.create_emergency_images()

    def create_simple_poster(self, title, image_name, genre, is_series=False):
        """Create a simple but nice poster"""
        try:
            width, height = 200, 300
            colors = ['#e50914', '#00a8ff', '#9c88ff', '#fbc531', '#4cd137', '#487eb0', '#e84118', '#8c7ae6']
            bg_color = random.choice(colors)
            
            img = Image.new('RGB', (width, height), color=bg_color)
            draw = ImageDraw.Draw(img)
            
            try:
                title_font = ImageFont.truetype("arial.ttf", 16)
                genre_font = ImageFont.truetype("arial.ttf", 12)
                tag_font = ImageFont.truetype("arial.ttf", 10)
            except:
                title_font = ImageFont.load_default()
                genre_font = ImageFont.load_default()
                tag_font = ImageFont.load_default()
            
            for i in range(50):
                y = height - 50 + i
                r, g, b = int(bg_color[1:3], 16), int(bg_color[3:5], 16), int(bg_color[5:7], 16)
                new_color = (max(0, r-10), max(0, g-10), max(0, b-10))
                hex_color = f"#{new_color[0]:02x}{new_color[1]:02x}{new_color[2]:02x}"
                draw.rectangle([0, y, width, y+1], fill=hex_color)
            
            words = title.split()
            lines = []
            current_line = []
            
            for word in words:
                test_line = ' '.join(current_line + [word])
                approx_width = len(test_line) * 10
                if approx_width <= width - 20:
                    current_line.append(word)
                else:
                    if current_line:
                        lines.append(' '.join(current_line))
                    current_line = [word]
            if current_line:
                lines.append(' '.join(current_line))
            
            title_y = 100 - (len(lines) * 10)
            for i, line in enumerate(lines):
                draw.text((width//2, title_y + i*20), line, fill="white", font=title_font, anchor="mm")
            
            draw.text((width//2, 180), genre.upper(), fill="white", font=genre_font, anchor="mm")
            
            badge_text = "SERIES" if is_series else "MOVIE"
            badge_color = "#ff6b6b" if is_series else "#74b9ff"
            
            badge_width = 80
            badge_height = 25
            badge_x = (width - badge_width) // 2
            badge_y = 220
            
            draw.rectangle([badge_x, badge_y, badge_x + badge_width, badge_y + badge_height], fill=badge_color)
            draw.text((width//2, badge_y + badge_height//2), badge_text, fill="white", font=tag_font, anchor="mm")
            
            draw.text((width//2, 260), "FROZZLE", fill="white", font=tag_font, anchor="mm")
            
            image_path = os.path.join(self.images_dir, f"{image_name}.jpg")
            img.save(image_path)
            
        except Exception as e:
            print(f"Error creating poster for {title}: {e}")
            self.create_basic_fallback(title, image_name)

    def create_basic_fallback(self, title, image_name):
        """Create absolute basic fallback image"""
        try:
            img = Image.new('RGB', (200, 300), color='#2a2a2a')
            draw = ImageDraw.Draw(img)
            
            if len(title) > 25:
                title = title[:22] + "..."
            
            if len(title) > 20:
                mid = len(title) // 2
                part1 = title[:mid]
                part2 = title[mid:]
                draw.text((100, 120), part1, fill="white", anchor="mm")
                draw.text((100, 140), part2, fill="white", anchor="mm")
            else:
                draw.text((100, 130), title, fill="white", anchor="mm")
            
            draw.text((100, 180), "FROZZLE", fill="#e50914", anchor="mm")
            
            image_path = os.path.join(self.images_dir, f"{image_name}.jpg")
            img.save(image_path)
            print(f"✓ Created fallback: {image_name}.jpg")
            
        except Exception as e:
            print(f"Critical error creating fallback for {title}: {e}")

    def create_emergency_images(self):
        """Create absolute minimum images if everything else fails"""
        print("Creating emergency fallback images...")
        colors = ['red', 'blue', 'green', 'purple', 'orange']
        
        all_content = self.get_movies_data() + self.get_series_data()
        for i, item in enumerate(all_content[:30]):
            try:
                img = Image.new('RGB', (200, 300), color=colors[i % len(colors)])
                draw = ImageDraw.Draw(img)
                draw.text((100, 150), item["name"][:15], fill="white", anchor="mm")
                image_path = os.path.join(self.images_dir, f"{item['image']}.jpg")
                img.save(image_path)
            except:
                pass

    def load_users(self):
        """Load users from JSON file"""
        try:
            if os.path.exists(self.data_file):
                with open(self.data_file, 'r') as f:
                    return json.load(f)
        except:
            pass
        return {}

    def save_users(self):
        """Save users to JSON file"""
        try:
            with open(self.data_file, 'w') as f:
                json.dump(self.users, f)
        except Exception as e:
            messagebox.showerror("Error", f"Could not save user data: {str(e)}")

    def hash_password(self, password):
        """Hash password for security"""
        return hashlib.sha256(password.encode()).hexdigest()

    def validate_username(self, username):
        """Validate username"""
        if len(username) < 3:
            return False, "Username must be at least 3 characters long"
        return True, ""

    def validate_password(self, password):
        """Validate password"""
        if len(password) < 6:
            return False, "Password must be at least 6 characters long"
        return True, ""

    def get_image_path(self, image_name):
        """Get platform-independent image path"""
        image_path = os.path.join(self.images_dir, f"{image_name}.jpg")
        if os.path.exists(image_path):
            return image_path
        return None

    def get_movies_data(self):
        """Get movies data - COMPLETE DATASET"""
        return [
            {"name": "URI: The Surgical Strike", "url": "https://www.cineby.gd/movie/554600", "image": "URI", "genre": "Action"},
            {"name": "Sitare zamee par", "url": "https://www.cineby.gd/movie/1190511", "image": "SZP", "genre": "comedy"},
            {"name": "Dangal", "url": "https://www.cineby.gd/movie/360814", "image": "dangal.webp", "genre": "Sports"},
            {"name": "3 Idiots", "url": "https://www.cineby.gd/movie/20453", "image": "3idots", "genre": "Comedy"},
            {"name": "Bahubali: The Beginning", "url": "https://www.cineby.gd/movie/256040", "image": "bahubali1", "genre": "Action"},
            {"name": "Bahubali: The Conclusion", "url": "https://www.cineby.gd/movie/350312", "image": "bahubali2", "genre": "Action"},
            {"name": "KGF: Chapter 1", "url": "https://www.cineby.gd/movie/564147", "image": "kgf1", "genre": "Action"},
            {"name": "KGF: Chapter 2", "url": "https://www.cineby.gd/movie/587412", "image": "kgf2", "genre": "Action"},
            {"name": "RRR", "url": "https://www.cineby.gd/movie/579974", "image": "rrr", "genre": "Action"},
            {"name": "Pushpa: The Rise", "url": "https://www.cineby.gd/movie/690957", "image": "pushpa", "genre": "Action"},
            {"name": "Vikram Vedha", "url": "https://www.cineby.gd/movie/432139", "image": "vikramvedha", "genre": "Action"},
            {"name": "War", "url": "https://www.cineby.gd/movie/585268", "image": "war", "genre": "Action"},
            {"name": "Kabir Singh", "url": "https://www.cineby.gd/movie/577328", "image": "kabirsingh", "genre": "Romance"},
            {"name": "Andhadhun", "url": "https://www.cineby.gd/movie/534780", "image": "andhadhun", "genre": "Thriller"},
            {"name": "Stree", "url": "https://www.cineby.gd/movie/533991", "image": "stree", "genre": "Comedy"},
            # Add more movies here to reach 100+
        ]

    def get_series_data(self):
        """Get series data - COMPLETE DATASET"""
        return [
            {"name": "Squid Game", "url": "https://www.cineby.gd/tv/93405", "image": "squidgame", "genre": "Thriller"},
            {"name": "Stranger Things", "url": "https://www.cineby.gd/tv/66732", "image": "strangerthings", "genre": "Sci-Fi"},
            {"name": "Money Heist", "url": "https://www.cineby.gd/tv/71446", "image": "moneyheist", "genre": "Crime"},
            {"name": "The Witcher", "url": "https://www.cineby.gd/tv/71912", "image": "thewitcher", "genre": "Fantasy"},
            {"name": "Breaking Bad", "url": "https://www.cineby.gd/tv/1396", "image": "breakingbad", "genre": "Drama"},
            {"name": "Game of Thrones", "url": "https://www.cineby.gd/tv/1399", "image": "gameofthrones", "genre": "Fantasy"},
            {"name": "The Mandalorian", "url": "https://www.cineby.gd/tv/82856", "image": "mandalorian", "genre": "Sci-Fi"},
            {"name": "Wednesday", "url": "https://www.cineby.gd/tv/119051", "image": "wednesday", "genre": "Comedy"},
            {"name": "Lucifer", "url": "https://www.cineby.gd/tv/63174", "image": "lucifer", "genre": "Crime"},
            {"name": "The Crown", "url": "https://www.cineby.gd/tv/65494", "image": "thecrown", "genre": "Drama"},
            {"name": "Mirzapur", "url": "https://www.cineby.gd/tv/84105", "image": "mirzapur", "genre": "Crime"},
            {"name": "Sacred Games", "url": "https://www.cineby.gd/tv/79352", "image": "sacredgames", "genre": "Thriller"},
            {"name": "The Family Man", "url": "https://www.cineby.gd/tv/93352", "image": "familyman", "genre": "Action"},
            {"name": "Panchayat", "url": "https://www.cineby.gd/tv/11352", "image": "panchayat", "genre": "Comedy"},
            {"name": "Kota Factory", "url": "https://www.cineby.gd/tv/89113", "image": "kotafactory", "genre": "Drama"},
            # Add more series here to reach 50+
        ]

# Create and run the application
if __name__ == "__main__":
    root = tk.Tk()
    app = MovieStreamingApp(root)
    root.mainloop()
