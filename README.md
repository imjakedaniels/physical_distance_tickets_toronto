# Physical Distance Tickets Toronto

<p align="center">
  <img src="https://github.com/imjakedaniels/physical_distance_tickets_toronto/blob/master/visuals/tickets_issued_plot.png">
</p>

Packages Used

```r
library(tidyverse) # basic manipulation
library(lubridate) # date manipulation
library(shadowtext) # easy white shadow behind bar labels
library(extrafont) # custom fonts
library(ggtext) # html stylized fonts
```

Ticket count is referring to tickets given by MLS officers.

Media text was extracted from [various media announcements](https://www.toronto.ca/home/media-room/news-releases-media-advisories/).

The appropriate excerpts were included below.

# Preface 

Mayor John Tory signed an emergency order No. 1 (April 1) and emergency order No. 2 (April 3) regulating physical distancing in City of Toronto parks and public squares. Any two people who don’t live together, who fail to keep two metres of distance between them in a park or public square, can receive a $1,000 ticket – the maximum set fine available. 

Officers could issue higher tickets that would be subject to the courts where fines could go up to $5,000 upon conviction. 

## Media Announcements

_I've added italicised dates into some of these statements to make "yesterday" specific._

### Pre-Emergency Order No. 2

> Between _March 24 and April 3rd,_ Municipal Licensing & Standards has responded to 407 complaints and issued 34 notices for failure to comply with the Province’s orders under the Emergency Management and Civil Protection Act. Toronto Police Services has issued **tickets to 21 people** for non-compliance, summonses to two businesses, and nine notices for failure to comply with provincial orders.

```r
pre_shutdown_data <- data.frame(
  date = c("2020-03-24", "2020-04-03"),
  tickets_issued = c(0, 21),
  weekend = c(F, F))
```

### Emergency Order No. 2 signed.

> Yesterday, _Saturday April 4th_, 311 received 141 complaints about gatherings and unsafe behaviour at parks. In just the first day of the enforcement blitz, 800 vehicles were turned away at Bluffers Park and 140 vehicles were deterred from parking at High Park. Police issued nine tickets while MLS officers gave out **one ticket** related to park amenities and **five** to non-essential businesses that were operating in violation of provincial orders. 

> We are _missing Sunday April 5th update._ Data will be deduced from the next announcements.

> The City also received 385 complaints yesterday, _Monday April 6th_, alone about people using closed park amenities or not practising physical distancing in parks. MLS spoke to 848 people regarding the closure of park amenities and distancing and issued **12 tickets** – the highest number of tickets issued by MLS in a single day since the start of the pandemic.

> Yesterday, _Tuesday April 7th_, the City received 482 complaints about people using park amenities or not practising physical distancing in parks. Bylaw officers spoke to 629 people regarding the closure of park amenities and distancing and issued 37 written cautions and **33 tickets** – bringing the total to 53 tickets since April 4. 

> _Therefore, 6 + x + 12 + 33 = 53 // x = 2 tickets given Sunday April 5th._

```r
first_check <- data.frame(date = c("2020-04-04", "2020-04-05", "2020-04-06",  "2020-04-07"),
                          daily_tickets_issued = c(6, 2, 12, 33)) %>%
  mutate(total_tickets_issued = cumsum(daily_tickets_issued))

first_check
```

> Yesterday, _Wednesday April 8th_, the City received 356 complaints involving people using amenities or not practising physical distancing in parks. Bylaw officers spoke to 989 people regarding the closure of park amenities and distancing and issued 16 written cautions and **15 tickets** – bringing the total to 68 tickets since April 4.

> Yesterday, _Thursday April 9th_, the City received 550 complaints involving people using amenities or not practising physical distancing in parks. Bylaw officers spoke to 770 people regarding the closure of park amenities and distancing, and issued 32 written cautions and **11 tickets** – bringing the total to 79 tickets since April 4.

```r
sanity_check <- data.frame(date = c("2020-04-04", "2020-04-05", "2020-04-06",  "2020-04-07", "2020-04-08", "2020-04-09"),
                           daily_tickets_issued = c(6, 2, 12, 33, 15, 11)) %>%
  mutate(total_tickets_issued = cumsum(daily_tickets_issued))

sanity_check
```

> _Missing Friday April 10th update. 

> Yesterday, _Saturday April 11th_, the team moved to almost exclusively issuing tickets. Municipal Licensing & Standards (MLS) officers issued **48 tickets** for the use of closed park amenities and not practising physical distancing – 32 per cent of the total number of tickets issued since enforcement began nine days ago on April 3.

> Yesterday, _Sunday April 12th_, Municipal Licensing & Standards (MLS) officers issued **40 tickets** for the use of closed park amenities and not practising physical distancing. This brings the total to 107 tickets issued over the long weekend, which accounts for 56 per cent of the total tickets issued since April 4.

> _Therefore, x + 48 + 40 = 107 // x = 19 tickets given Sunday April 10th._

```r
second_check <- data.frame(date = c("2020-04-10", "2020-04-11", "2020-04-12"),
                           daily_tickets_issued = c(19, 48, 40)) %>%
  mutate(total_tickets_issued = cumsum(daily_tickets_issued))

second_check
```

> Yesterday, _Monday April 13th_, 311 received 324 complaints related to activity in parks and bylaw enforcement officers issued **26 tickets**. More than 130 tickets were issued from Friday to Monday, 61 per cent of the 217 tickets issued since the start of enforcement on April 4.

> Yesterday, _Tuesday April 14th_ the City received 396 complaints involving people using amenities or not practising physical distancing in parks. Bylaw and police officers cautioned 470 individuals regarding the closure of park amenities and physical distancing, and issued **35 tickets* – bringing the total to 252 tickets since April 3. 

```r
shutdown_df <- data.frame(date = c("2020-04-14", "2020-04-13", "2020-04-12", "2020-04-11", "2020-04-10", "2020-04-09", "2020-04-08", "2020-04-07", "2020-04-06",  "2020-04-05", "2020-04-04"),
                          daily_tickets_issued = c(35, 26, 40, 48, 19, 11, 15, 33, 12, 2, 6),
                          weekend = c(F, F, T, T, F, F, F, F, F, T, T)) %>%
  mutate(total_tickets_issued = cumsum(daily_tickets_issued))

shutdown_df %>%
  tail(1)
```

> The city is now using April 3rd as their benchmark (previous raw totals referred to April 4th!) and we are short 5 tickets. So we can deduce 5 tickets were given April 3rd.

```r
april_3rd <- data.frame(date = "2020-04-03",
                        daily_tickets_issued = 5,
                        weekend = FALSE)

cleaned_shutdown_df <- april_3rd %>%
  bind_rows(shutdown_df) %>%
  mutate(date = as.Date(date)) %>%
  arrange(date) %>%
  mutate(total_tickets_issued = cumsum(daily_tickets_issued))
```

## Plotting

```r
p <- cleaned_shutdown_df %>%
  ggplot(aes(x = date, y = daily_tickets_issued, label = daily_tickets_issued)) +
  geom_col(aes(fill = weekend)) +
  
  geom_point(data = tail(cleaned_shutdown_df, 1), 
             aes(y = total_tickets_issued), colour = "orange") +
  geom_text(data = tail(cleaned_shutdown_df, 1), 
            aes(y = total_tickets_issued, label = total_tickets_issued), colour = "orange", vjust = -0.5, family = "IBM Plex Sans SemiBold") +
  
  geom_line(aes(y = total_tickets_issued), colour = "orange") +
  geom_shadowtext(aes(label = case_when(
    date == "2020-04-03" ~ "5*",
    date == "2020-04-05" ~ "2*",
    date == "2020-04-10" ~ "19*",
    TRUE ~ as.character(daily_tickets_issued))), colour = "black", bg.colour='white', vjust = -0.5, size = 3) +
  
  scale_x_date("", labels = scales::date_format("%a.\n %b-%d"), date_breaks = "1 day") +
  scale_fill_manual(values = c("blue", "red")) +
  expand_limits(y = 0:250) +
  coord_cartesian(clip = "off", xlim = c(as.Date("2020-04-03"), as.Date("2020-04-14"))) +
  
  theme_minimal() +
  theme(text = element_text(family = "IBM Plex Sans SemiBold"),
        plot.title = element_text(family = "IBM Plex Sans SemiBold", size = 12, hjust = 0),
        plot.subtitle =  element_markdown(family = "IBM Plex Sans", size = 8),
        plot.caption = element_text(family = "IBM Plex Sans", size = 8, hjust = 0, colour = "grey50"),
        axis.text.x = element_text(family = "IBM Plex Sans", size = 8),
        axis.title.y = element_text(family = "IBM Plex Sans", colour = "grey20", size = 8),
        panel.grid.minor = element_blank(),
        panel.grid.major.x = element_blank(),
        plot.background = element_rect(fill = "grey96"),
        legend.position = "none",
        plot.margin = unit(c(0.5, 0.5, 0.5, 0.5), units = "cm")) +
  
  labs(title = "Tickets Issued Since Regulating Physical Distancing\nin Toronto Parks and Public Squares",
       subtitle = "<b style='color:orange'>— Total Tickets</b> <b style='color:red'>◼︎ Weekends</b>",
       y = "Tickets Issued",
       caption = "SOURCE: News Releases & Media Advisories, City of Toronto\n*Deduced from data")
```


```r
ggsave(filename = "tickets_issued_plot.png",
       plot = p,
       dpi = 300,
       width = 7.2,
       height = 3)
```

