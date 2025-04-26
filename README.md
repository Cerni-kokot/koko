Can you insert something into this document and whislt makeing sure that the ptyhon works?? do it in the order i give you the code you need to insert at the correct palce ok. ::1. Modifications to Game State Variables
Add or modify these variables in your initialization section (where money, gang_members, etc., are defined) to support the new features:
python

# Add to existing variables
active_businesses = []  # List of dictionaries: {"type": "Gun Production", "country": "France"}
country_business = {country: None for country in countries}  # Tracks business type per country
profile_dialog = False
profile_member = None
profile_member_index = None
profile_member_country = None
relocate_business_dialog = False
business_to_relocate = None

Where to Place: Add these after your existing variable declarations (e.g., after country_gang_members).
2. Update Button Rectangles
Modify the pause button position and add the profile dialog buttons. Replace or add to your button rectangle definitions:
python

# Update pause button position
pause_button = pygame.Rect(10, HEIGHT - 50, 100, 40)  # Moved to bottom-left

# Add profile dialog buttons
profile_close_button = pygame.Rect(WIDTH // 2 + 20, HEIGHT // 2 + 50, 80, 40)
unassign_button = pygame.Rect(WIDTH // 2 - 100, HEIGHT // 2 + 50, 80, 40)

Where to Place: Replace the existing pause_button definition and add the profile buttons where other button rectangles are defined (e.g., gang_button, bank_button).
3. Update Event Handling for New Features
Modify the event handling section in the main game loop to include logic for the profile dialog, business relocation, and updated gang member interactions. Replace the relevant part of the event handling (inside for event in pygame.event.get()):
python

elif event.type == pygame.MOUSEBUTTONDOWN:
    mouse_pos = pygame.mouse.get_pos()
    if pause_button.collidepoint(mouse_pos):
        paused = not paused
        message = "Paused" if paused else "Resumed"
        message_timer = current_time
    if paused:
        continue

    # Existing confirmation dialogs (buy country, buy/sell gang member, etc.)
    if confirmation_active:
        # ... (keep existing code for country purchase confirmation)
    elif buy_gang_first_confirm:
        # ... (keep existing code for gang member purchase first confirmation)
    elif buy_gang_second_confirm:
        # ... (keep existing code for gang member purchase second confirmation)
    elif sell_gang_confirm:
        # ... (keep existing code for gang member sell confirmation)
    elif cancel_business_confirm:
        # ... (keep existing code for business cancel confirmation)
    elif relocate_business_dialog:
        dialog_rect = pygame.Rect(WIDTH // 2 - 200, HEIGHT // 2 - 100, 400, 200)
        owned_countries = [country for country, data in countries.items() if data["owned"] and country_business[country] is None]
        for i, country in enumerate(owned_countries):
            country_button = pygame.Rect(WIDTH // 2 - 180, HEIGHT // 2 - 60 + i * 30, 360, 25)
            if country_button.collidepoint(mouse_pos):
                if business_to_relocate is not None:
                    business = active_businesses[business_to_relocate]
                    old_country = business["country"]
                    country_business[old_country] = None
                    countries[old_country]["business_income"] = 0
                    business["country"] = country
                    country_business[country] = business["type"]
                    num_members = len(country_gang_members[country])
                    base_income = BUSINESS_INCOME_RATES[business["type"]]
                    countries[country]["business_income"] = int(base_income * (1 + num_members * 0.1))
                    message = f"Relocated {business['type']} to {country}!"
                    message_timer = current_time
                relocate_business_dialog = False
                business_to_relocate = None
                break
        if not dialog_rect.collidepoint(mouse_pos):
            relocate_business_dialog = False
            business_to_relocate = None
    elif profile_dialog:
        if profile_close_button.collidepoint(mouse_pos):
            profile_dialog = False
            profile_member = None
            profile_member_index = None
            profile_member_country = None
        elif unassign_button.collidepoint(mouse_pos) and profile_member_country:
            member = country_gang_members[profile_member_country].pop(profile_member_index)
            gang_members.append(member)
            num_members = len(country_gang_members[profile_member_country])
            if country_business[profile_member_country]:
                base_income = BUSINESS_INCOME_RATES[country_business[profile_member_country]]
                countries[profile_member_country]["business_income"] = int(base_income * (1 + num_members * 0.1))
            message = f"Unassigned {member} from {profile_member_country}!"
            message_timer = current_time
            profile_dialog = False
            profile_member = None
            profile_member_index = None
            profile_member_country = None
    elif choose_business_dialog:
        # ... (keep existing business selection code, but update to use active_businesses)
        if gun_button.collidepoint(mouse_pos):
            if money >= 500:
                money -= 500
                selected_business = "Gun Production"
                assign_business_dialog = True
                business_to_assign = selected_business
                message = "Selected Gun Production business! $500 deducted."
                message_timer = current_time
            else:
                message = "Need at least $500 to start a business!"
                message_timer = current_time
            choose_business_dialog = False
        # ... (similar updates for other business types)
    elif assign_business_dialog:
        dialog_rect = pygame.Rect(WIDTH // 2 - 200, HEIGHT // 2 - 100, 400, 200)
        owned_countries = [country for country, data in countries.items() if data["owned"] and country_business[country] is None]
        for i, country in enumerate(owned_countries):
            country_button = pygame.Rect(WIDTH // 2 - 180, HEIGHT // 2 - 60 + i * 30, 360, 25)
            if country_button.collidepoint(mouse_pos):
                if business_to_assign:
                    country_business[country] = business_to_assign
                    num_members = len(country_gang_members[country])
                    base_income = BUSINESS_INCOME_RATES[business_to_assign]
                    countries[country]["business_income"] = int(base_income * (1 + num_members * 0.1))
                    active_businesses.append({"type": business_to_assign, "country": country})
                    message = f"Assigned {business_to_assign} to {country}!"
                    message_timer = current_time
                assign_business_dialog = False
                business_to_assign = None
                selected_business = None
                break
        if not dialog_rect.collidepoint(mouse_pos):
            assign_business_dialog = False
            business_to_assign = None
            selected_business = None
    elif gang_panel_active:
        panel_x, panel_y = gang_panel_pos
        gang_panel_rect = pygame.Rect(panel_x + 50, panel_y, 250, 300)
        members_tab_button = pygame.Rect(panel_x, panel_y, 50, 30)
        business_tab_button = pygame.Rect(panel_x, panel_y + 30, 50, 30)
        # ... (other tab buttons)
        gang_close_button = pygame.Rect(panel_x + 220, panel_y + 270, 80, 30)
        if gang_close_button.collidepoint(mouse_pos):
            gang_panel_active = False
            current_gang_tab = "Members"
        elif members_tab_button.collidepoint(mouse_pos):
            current_gang_tab = "Members"
        elif business_tab_button.collidepoint(mouse_pos):
            current_gang_tab = "Business"
        elif current_gang_tab == "Members":
            if buy_gang_member_button.collidepoint(mouse_pos):
                buy_gang_first_confirm = True
            all_members = []
            for i, member in enumerate(gang_members):
                all_members.append((member, None, i))
            for country in countries:
                for i, member in enumerate(country_gang_members[country]):
                    all_members.append((member, country, i))
            for i, (member, country, index) in enumerate(all_members):
                profile_button = pygame.Rect(panel_x + 270, panel_y + 70 + i * 20, 20, 20)
                assign_button = pygame.Rect(panel_x + 170, panel_y + 70 + i * 20, 90, 20)
                if profile_button.collidepoint(mouse_pos):
                    profile_dialog = True
                    profile_member = member
                    profile_member_index = index
                    profile_member_country = country
                    break
                elif assign_button.collidepoint(mouse_pos) and not country:
                    assign_member_dialog = True
                    member_to_assign = index
                    break
        elif current_gang_tab == "Business":
            if choose_business_button.collidepoint(mouse_pos):
                choose_business_dialog = True
            for i, business in enumerate(active_businesses):
                relocate_button = pygame.Rect(panel_x + 170, panel_y + 90 + i * 20, 90, 20)
                cancel_button = pygame.Rect(panel_x + 270, panel_y + 90 + i * 20, 20, 20)
                if relocate_button.collidepoint(mouse_pos):
                    relocate_business_dialog = True
                    business_to_relocate = i
                    break
                elif cancel_button.collidepoint(mouse_pos):
                    cancel_business_confirm = True
                    cancel_business_index = i
                    break
        # ... (keep existing code for other tabs and panel close)
    # ... (keep existing code for other conditions)

Where to Place: Replace the existing MOUSEBUTTONDOWN event handling section in your main game loop. Ensure you merge with any existing dialogs (e.g., country purchase, borrow money) that I marked with # ....
4. Update Business Income Logic
Modify the business income update logic to use active_businesses and country_business. Replace the income update section in the main loop (just before rendering):
python

if not paused:
    current_time = time.time()
    if current_time - last_business_income_time >= 1:
        for country, data in countries.items():
            if data["owned"] and country_business[country]:
                income = data["business_income"]
                money += income
        last_business_income_time = current_time
    date_str = update_date(current_time)

Where to Place: Replace the existing income update logic in the main loop (likely where money is incremented based on business income).
5. Update Upper Main Panel with Income Sources
Modify the upper middle panel rendering to include income source tooltips. Replace the tooltip rendering part (inside the upper panel rendering):
python

# Inside the upper middle panel rendering
tooltip_text = None
tooltip_x = 0
if col1_rect.collidepoint(mouse_pos):
    income_sources = []
    for business in active_businesses:
        country = business["country"]
        income = countries[country]["business_income"]
        income_sources.append(f"{business['type']} in {country}: ${income}/s")
    tooltip_text = f"Money: ${money}"
    if income_sources:
        tooltip_text += "\nIncome from:\n" + "\n".join(income_sources)
    else:
        tooltip_text += "\nNo income sources"
    tooltip_x = col1_x + col_width // 2
elif col2_rect.collidepoint(mouse_pos):
    tooltip_text = f"Bank Debt: ${bank_debt}"
    tooltip_x = col2_x + col_width // 2
elif col3_rect.collidepoint(mouse_pos):
    tooltip_text = f"Reputation: {reputation}"
    tooltip_x = col3_x + col_width // 2
elif col4_rect.collidepoint(mouse_pos):
    tooltip_text = f"Countries Owned: {sum(1 for data in countries.values() if data['owned'])}/{len(countries)}"
    tooltip_x = col4_x + col_width // 2
elif col5_rect.collidepoint(mouse_pos):
    tooltip_text = f"Date: {date_str}"
    tooltip_x = col5_x + col_width // 2

if tooltip_text:
    lines = tooltip_text.split('\n')
    max_width = max(font.size(line)[0] for line in lines)
    total_height = len(lines) * font.get_height()
    tooltip_rect = pygame.Rect(tooltip_x - max_width // 2, 75, max_width, total_height)
    tooltip_bg_rect = tooltip_rect.inflate(10, 10)
    pygame.draw.rect(screen, WHITE, tooltip_bg_rect)
    pygame.draw.rect(screen, BLACK, tooltip_bg_rect, 1)
    for i, line in enumerate(lines):
        tooltip_surface = font.render(line, True, BLACK)
        screen.blit(tooltip_surface, (tooltip_x - font.size(line)[0] // 2, 75 + i * font.get_height()))

Where to Place: Replace the tooltip rendering code in the upper middle panel section (where stats_rect is drawn).
6. Update Country Panel with Business Name
Add the business name to the country panel rendering. Modify the country panel rendering section:
python

if panel_active and panel_country:
    try:
        panel_rect = pygame.Rect(500, 150, 250, 230)
        pygame.draw.rect(screen, WHITE, panel_rect)
        pygame.draw.rect(screen, BLACK, panel_rect, 2)
        data = countries[panel_country]
        name_text = title_font.render(panel_country, True, BLACK)
        pop_text = font.render(f"Population: {data['population']:,}", True, BLACK)
        gang_text = font.render(f"Gang Members: {len(country_gang_members[panel_country])}", True, BLACK)
        income_text = font.render(f"Business Income: ${data['business_income']}/s", True, BLACK)
        business_text = font.render(f"Business: {country_business[panel_country] if country_business[panel_country] else 'None'}", True, BLACK)
        screen.blit(name_text, (510, 160))
        screen.blit(pop_text, (510, 200))
        screen.blit(gang_text, (510, 230))
        screen.blit(income_text, (510, 260))
        screen.blit(business_text, (510, 290))
        pygame.draw.rect(screen, GRAY, panel_close_button)
        pygame.draw.rect(screen, BLACK, panel_close_button, 2)
        close_text = font.render("Close", True, BLACK)
        close_rect = close_text.get_rect(center=panel_close_button.center)
        screen.blit(close_text, close_rect)
        if not data["owned"]:
            pygame.draw.rect(screen, GRAY, panel_buy_button)
            pygame.draw.rect(screen, BLACK, panel_buy_button, 2)
            buy_text = font.render("Buy", True, BLACK)
            buy_rect = buy_text.get_rect(center=panel_buy_button.center)
            screen.blit(buy_text, buy_rect)
    except Exception as e:
        print(f"Error rendering panel for {panel_country}: {e}")

Where to Place: Replace the existing country panel rendering code (where panel_active is checked).
7. Update Gang Panel for Members and Business
Modify the gang panel rendering to show all members and active businesses with relocation options. Replace the gang panel rendering section:
python

if gang_panel_active:
    try:
        panel_x, panel_y = gang_panel_pos
        members_tab_button = pygame.Rect(panel_x, panel_y, 50, 30)
        business_tab_button = pygame.Rect(panel_x, panel_y + 30, 50, 30)
        location_tab_button = pygame.Rect(panel_x, panel_y + 60, 50, 30)
        vehicle_tab_button = pygame.Rect(panel_x, panel_y + 90, 50, 30)
        diplomacy_tab_button = pygame.Rect(panel_x, panel_y + 120, 50, 30)
        gang_panel_rect = pygame.Rect(panel_x + 50, panel_y, 250, 300)
        pygame.draw.rect(screen, WHITE, gang_panel_rect)
        pygame.draw.rect(screen, BLACK, gang_panel_rect, 2)
        pygame.draw.rect(screen, GRAY if current_gang_tab == "Members" else WHITE, members_tab_button)
        pygame.draw.rect(screen, BLACK, members_tab_button, 1)
        members_text = font.render("M", True, BLACK)
        screen.blit(members_text, members_text.get_rect(center=members_tab_button.center))
        pygame.draw.rect(screen, GRAY if current_gang_tab == "Business" else WHITE, business_tab_button)
        pygame.draw.rect(screen, BLACK, business_tab_button, 1)
        business_text = font.render("B", True, BLACK)
        screen.blit(business_text, business_text.get_rect(center=business_tab_button.center))
        pygame.draw.rect(screen, GRAY if current_gang_tab == "Location" else WHITE, location_tab_button)
        pygame.draw.rect(screen, BLACK, location_tab_button, 1)
        location_text = font.render("L", True, BLACK)
        screen.blit(location_text, location_text.get_rect(center=location_tab_button.center))
        pygame.draw.rect(screen, GRAY if current_gang_tab == "Vehicle" else WHITE, vehicle_tab_button)
        pygame.draw.rect(screen, BLACK, vehicle_tab_button, 1)
        vehicle_text = font.render("V", True, BLACK)
        screen.blit(vehicle_text, vehicle_text.get_rect(center=vehicle_tab_button.center))
        pygame.draw.rect(screen, GRAY if current_gang_tab == "Gang Diplomacy" else WHITE, diplomacy_tab_button)
        pygame.draw.rect(screen, BLACK, diplomacy_tab_button, 1)
        diplomacy_text = font.render("D", True, BLACK)
        screen.blit(diplomacy_text, diplomacy_text.get_rect(center=diplomacy_tab_button.center))
        # ... (keep existing tooltip code for tabs)
        if current_gang_tab == "Members":
            buy_gang_member_button = pygame.Rect(panel_x + 60, panel_y + 30, 230, 30)
            pygame.draw.rect(screen, GRAY if money >= 150 else DARK_GRAY, buy_gang_member_button)
            pygame.draw.rect(screen, BLACK, buy_gang_member_button, 1)
            buy_text = font.render("Buy a New Gang Member ($150)", True, BLACK)
            buy_rect = buy_text.get_rect(center=buy_gang_member_button.center)
            screen.blit(buy_text, buy_rect)
            all_members = []
            for i, member in enumerate(gang_members):
                all_members.append((member, None, i))
            for country in countries:
                for i, member in enumerate(country_gang_members[country]):
                    all_members.append((member, country, i))
            for i, (member, country, index) in enumerate(all_members):
                member_text = font.render(f"{member} ({country if country else 'Unassigned'})", True, BLACK)
                screen.blit(member_text, (panel_x + 60, panel_y + 70 + i * 20))
                profile_button = pygame.Rect(panel_x + 270, panel_y + 70 + i * 20, 20, 20)
                assign_button = pygame.Rect(panel_x + 170, panel_y + 70 + i * 20, 90, 20)
                pygame.draw.rect(screen, GRAY, profile_button)
                pygame.draw.rect(screen, BLACK, profile_button, 1)
                profile_text = font.render("P", True, BLACK)
                profile_rect = profile_text.get_rect(center=profile_button.center)
                screen.blit(profile_text, profile_rect)
                if not country:
                    pygame.draw.rect(screen, GRAY, assign_button)
                    pygame.draw.rect(screen, BLACK, assign_button, 1)
                    assign_text = font.render("Assign", True, BLACK)
                    assign_rect = assign_text.get_rect(center=assign_button.center)
                    screen.blit(assign_text, assign_rect)
        elif current_gang_tab == "Business":
            choose_business_button = pygame.Rect(panel_x + 60, panel_y + 30, 230, 30)
            pygame.draw.rect(screen, GRAY, choose_business_button)
            pygame.draw.rect(screen, BLACK, choose_business_button, 1)
            choose_text = font.render("Start New Business", True, BLACK)
            choose_rect = choose_text.get_rect(center=choose_business_button.center)
            screen.blit(choose_text, choose_rect)
            if active_businesses:
                business_listny_list_text = font.render("Active Businesses:", True, BLACK)
                screen.blit(business_list_text, (panel_x + 60, panel_y + 60))
                for i, business in enumerate(active_businesses):
                    business_text = font.render(f"{business['type']} in {business['country']}", True, BLACK)
                    screen.blit(business_text, (panel_x + 60, panel_y + 90 + i * 20))
                    relocate_button = pygame.Rect(panel_x + 170, panel_y + 90 + i * 20, 90, 20)
                    cancel_button = pygame.Rect(panel_x + 270, panel_y + 90 + i * 20, 20, 20)
                    pygame.draw.rect(screen, GRAY, relocate_button)
                    pygame.draw.rect(screen, BLACK, relocate_button, 1)
                    relocate_text = font.render("Relocate", True, BLACK)
                    relocate_rect = relocate_text.get_rect(center=relocate_button.center)
                    screen.blit(relocate_text, relocate_rect)
                    pygame.draw.rect(screen, RED, cancel_button)
                    pygame.draw.rect(screen, BLACK, cancel_button, 1)
                    cancel_text = font.render("X", True, BLACK)
                    cancel_rect = cancel_text.get_rect(center=cancel_button.center)
                    screen.blit(cancel_text, cancel_rect)
        elif current_gang_tab == "Location":
            # ... (keep existing location tab code)
        elif current_gang_tab in ["Vehicle", "Gang Diplomacy"]:
            placeholder_text = font.render(f"{current_gang_tab} - Coming Soon", True, BLACK)
            screen.blit(placeholder_text, (panel_x + 60, panel_y + 30))
        gang_close_button = pygame.Rect(panel_x + 220, panel_y + 270, 80, 30)
        pygame.draw.rect(screen, GRAY, gang_close_button)
        pygame.draw.rect(screen, BLACK, gang_close_button, 2)
        close_text = font.render("Close", True, BLACK)
        close_rect = close_text.get_rect(center=gang_close_button.center)
        screen.blit(close_text, close_rect)
    except Exception as e:
        print(f"Error rendering gang panel: {e}")

Where to Place: Replace the existing gang panel rendering code (where gang_panel_active is checked).
8. Add Profile Dialog Rendering
Add the rendering code for the gang member profile dialog. Insert this after the gang panel rendering but before any other dialogs:
python

if profile_dialog and profile_member:
    try:
        overlay = pygame.Surface((WIDTH, HEIGHT), pygame.SRCALPHA)
        overlay.fill((0, 0, 0, 128))
        screen.blit(overlay, (0, 0))
        dialog_rect = pygame.Rect(WIDTH // 2 - 200, HEIGHT // 2 - 100, 400, 200)
        pygame.draw.rect(screen, WHITE, dialog_rect)
        pygame.draw.rect(screen, BLACK, dialog_rect, 2)
        title_text = dialog_font.render(f"Profile: {profile_member}", True, BLACK)
        location_text = dialog_font.render(f"Location: {profile_member_country if profile_member_country else 'Unassigned'}", True, BLACK)
        title_rect = title_text.get_rect(center=(WIDTH // 2, HEIGHT // 2 - 50))
        location_rect = location_text.get_rect(center=(WIDTH // 2, HEIGHT // 2 - 20))
        screen.blit(title_text, title_rect)
        screen.blit(location_text, location_rect)
        pygame.draw.rect(screen, GRAY, profile_close_button)
        pygame.draw.rect(screen, BLACK, profile_close_button, 2)
        close_text = font.render("Close", True, BLACK)
        close_rect = close_text.get_rect(center=profile_close_button.center)
        screen.blit(close_text, close_rect)
        if profile_member_country:
            pygame.draw.rect(screen, GRAY, unassign_button)
            pygame.draw.rect(screen, BLACK, unassign_button, 2)
            unassign_text = font.render("Unassign", True, BLACK)
            unassign_rect = unassign_text.get_rect(center=unassign_button.center)
            screen.blit(unassign_text, unassign_rect)
    except Exception as e:
        print(f"Error rendering profile dialog: {e}")

Where to Place: Insert this code after the gang panel rendering section but before other dialog rendering (e.g., buy_gang_first_confirm).
9. Add Relocate Business Dialog Rendering
Add the rendering code for the business relocation dialog. Insert this after the profile dialog rendering:
python

if relocate_business_dialog:
    try:
        overlay = pygame.Surface((WIDTH, HEIGHT), pygame.SRCALPHA)
        overlay.fill((0, 0, 0, 128))
        screen.blit(overlay, (0, 0))
        dialog_rect = pygame.Rect(WIDTH // 2 - 200, HEIGHT // 2 - 100, 400, 200)
        pygame.draw.rect(screen, WHITE, dialog_rect)
        pygame.draw.rect(screen, BLACK, dialog_rect, 2)
        title_text = dialog_font.render("Relocate Business", True, BLACK)
        title_rect = title_text.get_rect(center=(WIDTH // 2, HEIGHT // 2 - 70))
        screen.blit(title_text, title_rect)
        owned_countries = [country for country, data in countries.items() if data["owned"] and country_business[country] is None]
        if not owned_countries:
            no_countries_text = dialog_font.render("No available countries!", True, BLACK)
            no_countries_rect = no_countries_text.get_rect(center=(WIDTH // 2, HEIGHT // 2))
            screen.blit(no_countries_text, no_countries_rect)
        else:
            for i, country in enumerate(owned_countries):
                country_button = pygame.Rect(WIDTH // 2 - 180, HEIGHT // 2 - 60 + i * 30, 360, 25)
                pygame.draw.rect(screen, GRAY, country_button)
                pygame.draw.rect(screen, BLACK, country_button, 1)
                country_text = font.render(country, True, BLACK)
                country_rect = country_text.get_rect(center=country_button.center)
                screen.blit(country_text, country_rect)
    except Exception as e:
        print(f"Error rendering relocate business dialog: {e}")

Where to Place: Insert this code after the profile dialog rendering section.
10. Update Pause Button Rendering
Modify the pause button rendering to include an icon. Replace the pause button rendering code:
python

try:
    button_color = DARK_GRAY if pause_button.collidepoint(mouse_pos) else GRAY
    pygame.draw.rect(screen, button_color, pause_button)
    pygame.draw.rect(screen, BLACK, pause_button, 1)
    pause_text = font.render("Pause" if not paused else "Resume", True, BLACK)
    text_rect = pause_text.get_rect(center=(pause_button.centerx + 10, pause_button.centery))
    screen.blit(pause_text, text_rect)
    if not paused:
        pygame.draw.rect(screen, BLACK, (pause_button.x + 10, pause_button.y + 10, 5, 20))
        pygame.draw.rect(screen, BLACK, (pause_button.x + 20, pause_button.y + 10, 5, 20))
    else:
        pygame.draw.polygon(screen, BLACK, [
            (pause_button.x + 10, pause_button.y + 10),
            (pause_button.x + 25, pause_button.y + 20),
            (pause_button.x + 10, pause_button.y + 30)
        ])
except Exception as e:
    print(f"Error rendering pause button: {e}")

Where to Place: Replace the existing pause button rendering code (where pause_button is drawn).

