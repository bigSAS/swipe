# swipe

```
import logging
from collections import namedtuple
from inpostqa_wtf import Actions
from typing import Tuple


logger = logging.getLogger(__name__)


Point = namedtuple('Point', 'x y')


def get_shaq(screensize: dict) -> Point:
    """
    Zwraca punkt srodka (centra) ekranu 👀
    """
    return Point(int(screensize['width'] / 2), int(screensize['height'] / 2))


def absolute(screensize: dict, option: str) -> Tuple[Point, Point]:
    if option in ('left', 'right'):
        center = Point(0, int(screensize['height'] / 2)) if option == 'right' \
            else Point(int(screensize['width']), int(screensize['height'] / 2))
        end = screensize['width'] if option == 'right' else 0
        return (center, Point(end, center.y))

    elif option in ('bottom', 'top'):
        center = Point(int(screensize['width'] / 2), int(screensize['height'] - 100)) if option == 'bottom' \
            else Point(int(screensize['width'] / 2), 0)
        end = 0 if option == 'bottom' else screensize['height']
        return (center, Point(center.x, end))


def pixalated(screensize: dict, pixels: int, option: str, starting_point: Point = None) -> Tuple[Point, Point]:
    center = starting_point if starting_point else get_shaq(screensize)
    if option in ('left', 'right'):
        end = center.x + pixels if option == 'right' else center.x - pixels
        return (center, Point(end, center.y))

    elif option in ('bottom', 'top'):
        end = center.y + pixels if option == 'bottom' else center.y - pixels
        return (center, Point(center.x, end))


def percented(screensize: dict, percent: int, option: str, starting_point: Point = None) -> Tuple[Point, Point]:
    def percented_pixels():
        dimension = 'height' if option in ('bottom', 'top') else 'width'
        h = screensize[dimension]
        m = (percent / 100)
        print(h, m)
        return int((h * m) / 2)

    center = starting_point if starting_point else get_shaq(screensize)
    if option in ('left', 'right'):
        end = center.x + percented_pixels() if option == 'right' else center.x - percented_pixels()
        return (center, Point(end, center.y))
    elif option in ('bottom', 'top'):
        end = center.y + percented_pixels() if option == 'bottom' else center.y - percented_pixels()
        return (center, Point(center.x, end))


def get_points(screensize: dict, instruction: str, starting_point: Point = None) -> Tuple[Point, Point]:
    """
    Oblicz punty do swipeowania
    Example instructions:
       * bottom:500px
       * bottom:abs
       * bottom:20%
       * top:500px
       * top:abs
       * top:20%
       * right:20px
       * right:20%
       * left:30px
       * left:10%

    """
    spl = instruction.split(':')
    option = spl[0]
    if option not in ['top', 'bottom', 'left', 'right']: raise ValueError(f'Invalid option: "{option}" fool')
    value = spl[1]

    if value == 'abs': return absolute(screensize, option)
    if 'px' in value: return pixalated(screensize, int(value.replace('px', '')), option, starting_point)
    if '%' in value: return percented(screensize, int(value.replace('%', '')), option, starting_point)

    raise ValueError(f'Ivalid instrucion: "{instruction}" U noob !')


def swipe(actions: Actions, instruction: str, duration_ms: int = 1000, override_start_point: Point = None):
    """
    todo: @tmajk -> migrate -> actions.swipe(instruction, timeout_ms)
    Example instructions:
        * bottom:500px - px relatywnie od srodka ekranu
                         (uwazac zeby nie podac za duzo px w stosunku do rozdzielczosci)
        * bottom:abs   - %  relatywnie od srodka ekranu
                         (50% to 1/4 ekranu 100% to 1/2 ekranu)
        * bottom:20%   - full od gory do dolu i odwrotnie
                         (0 -> H | H -> 0) (H - wysokosc ekranu)

        * top:500px
        * top:abs
        * top:20%

    Explanation override_start_point:
        jeśli chcemy, zeby zaczęło swipe z danego punktu to podajemy override_start_point w formie:
        * Point(x= 100, y=100)
        Jak nie podamy override_start_point to leci od środka ekranu

        todo: metodka, która bedzie zwracać współrzędne danego elementu

    Example:
    from lib import swipe
        * swipe(actions, 'bottom:500px') -> od srodka w dol 500px
        * swipe(actions, 'bottom:50%', 3000) -> od srodka w tol 50% (1/4 ekranu) + czas trwania swipe 3000ms
        * swipe(actions, 'top:abs') -> od 0 -> H (H - wysokosc ekranu)
        * swipe(
            actions=actions,
            instruction='right:200px',
            override_start_point=Point(x=500,y=500)
        ) -> swipe w prawo 200 px od punktu 500,500
    """
    logger.info(f'swipe: instruction: {instruction}, duration: {duration_ms} ms')
    screen_size = actions.webdriver.get_window_size()
    if override_start_point:
        start_point, end_point = get_points(
            screensize=screen_size,
            instruction=instruction,
            starting_point=override_start_point
        )
    else:
        start_point, end_point = get_points(screensize=screen_size, instruction=instruction)
    logger.info(f'start swipe: {start_point}, end swipe: {end_point}')
    actions.webdriver.swipe(start_point.x, start_point.y, end_point.x, end_point.y, duration_ms)
```
