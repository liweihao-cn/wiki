# UTF-8转Unicode

```c
#include <stdio.h>
#include <stdlib.h>

static const char head_mask[] = {0xff, 0x7f, 0x1f, 0x0f, 0x07, 0x03, 0x01};
static const char shift_tab[] = {0, 6, 12, 18, 24, 30};

int32_t utf8_cov_unicode(const char *stream)
{
        char head = *stream;
        char unicode_buf[4] = {0};
        int32_t *unicode = (int32_t *)unicode_buf;
        int length = 0;

        if (!(head&0x80))
                length = 1;
        else {
                while(head&0x80) {
                        length++;
                        head <<= 1;
                }
        }

        if (length == 1)
                unicode_buf[0] = head;
        else {
                stream += (length - 1);
                switch(length) {
                case 6:
                        *unicode |= (int32_t)((*stream--)&0x3f);
                case 5:
                        *unicode |= (int32_t)((*stream--)&0x3f) << shift_tab[length-5];
                case 4:
                        *unicode |= (int32_t)((*stream--)&0x3f) << shift_tab[length-4];
                case 3:
                        *unicode |= (int32_t)((*stream--)&0x3f) << shift_tab[length-3];
                case 2:
                        *unicode |= (int32_t)((*stream--)&0x3f) << shift_tab[length-2];
                        *unicode |= (int32_t)(*stream&head_mask[length-1]) << shift_tab[length-1];
                        break;
                default:
                        *unicode = -1;
                }
        }

        return *unicode;
}

int main(int argc, char *argv[])
{
        printf("%x\n", utf8_cov_unicode(argv[1]));
        return 0;
}
```
