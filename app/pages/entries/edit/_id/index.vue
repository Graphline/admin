<script>
import entryQuery from './entry.gql'
import updateEntryMutation from './update-entry.gql'
import vueMce from '~/components/core/mce-editor'
import slug from 'limax'

export default {
  validate ({route,}) {
    return route.params.id
  },
  'components': {
    vueMce,
  },
  async asyncData ({error, app, params,}) {
    const {data,} = await app.apolloProvider.defaultClient.query({
      'query':     entryQuery,
      'variables': {
        'id': params.id,
      },
    })

    if (!data.entry) {
      return error({
        'statusCode': 404,
        'message':    'Entry not found',
      })
    }

    return {
      'entry': data.entry,
    }
  },
  data () {
    return {
      'saving':     false,
      'leaving':    false,
      'slugLocked': false,
    }
  },
  beforeRouteLeave (to, from, next) {
    this.leaving = true
    const wait = new Promise((resolve) => setTimeout(resolve, 150))

    return wait.then(next)
  },
  'methods': {
    getHeight () {
      if (process.server) {
        return 403
      }

      return window.innerHeight - 263
    },
    async saveEntry () {
      this.saving = true

      try {
        await this.$apollo.mutate({
          'mutation':  updateEntryMutation,
          'variables': {
            'data': {
              'id':        this.entry.id,
              'display':   this.entry.display,
              'content':   this.entry.content,
              'published': true,
            },
          },
        })
      } catch (error) {
        this.saving = false
        return null
      }

      this.saving = false
    },
    updateSlug (value) {
      this.entry.slug = slug(value)
    },
  },
  'computed': {
    changed () {
      return true
    },
    defaultSlug () {
      return slug(this.entry.display)
    },
  },
}
</script>

<template lang="pug">
.route-root.is-paddingless
  v-layout(row)
    v-flex(xs12)
      v-card
        v-card-title.primary-title
          h2.title Editing entry

          v-spacer

          v-tooltip(left)
            v-btn(
              slot="activator", color="primary", fab, small
              aria-labelledby="entry-editor-create-new"
            )
              v-icon create
            span#entry-editor-create-new
              | Create new entry

  v-container
    v-layout(row, reverse, wrap)
      v-flex(xs12, sm12, md12, lg4, xl3)
        v-expansion-panel(popout, :value="0")
          v-expansion-panel-content(ripple)
            div(slot="header")
              v-layout(row)
                v-icon publish
                h4 Publication
            v-card
              v-card-text.px-4.py-0
                v-switch(
                  label="Published"
                  v-model="entry.published"
                )
              v-divider
              v-card-actions
                v-btn(color="red", fab, small, flat).white--text
                  v-icon delete
                v-spacer
                transition(name="zoom")
                  v-btn(
                    v-show="changed"
                    @click="saveEntry"
                    fab
                    color="primary"
                    small
                    :loading="saving"
                  )
                    v-icon save

          v-expansion-panel-content(ripple)
            h4(slot="header") Categories
            v-card.px-4.py-2.pt-4
              v-card-text
                p Text!

          v-expansion-panel-content(ripple)
            h4(slot="header") Featured image
            v-card.px-4.py-2.pt-4
              p HELLOE

          v-expansion-panel-content(ripple).red.white--text
            h4(slot="header") Danger
            v-card.px-4.py-2.pt-4
              p HELLOE

      v-flex(xs12, sm12, md12, lg8, xl9)
        v-card
          v-card-title.primary-title
            v-layout(column)
              h3.heading Metadata
              h4.subheading Edit post details, like the title and URL
          v-divider
          v-card-text
            form
              v-text-field(
                v-validate="'required'"
                v-model="entry.display"
                :error-messages="errors.collect('display')"
                label="Title"
                data-vv-name="display"
                required
                @input="updateSlug"
              )

              v-layout(row)
                v-text-field(
                  v-validate="'required'"
                  v-model="entry.slug"
                  :error-messages="errors.collect('slug')"
                  label="Slug"
                  data-vv-name="slug"
                  required
                )

                v-switch(v-model="slugLocked", label="Lock slug")

              v-datetime-picker(
                label="Publication date"
                v-model="entry.publishedAt"
                format="MMMM Do, YYYY - HH:mm"
              )

              //- v-dialog(
              //-   v-model="modal"
              //-   full-width
              //-   width="290px"
              //- )
              //-   v-text-field(
              //-     slot="activator"
              //-     v-model="entry.publishedAt"
              //-     label="Publication date"
              //-   )

        v-card
          v-card-title.primary-title
            v-layout(column)
              h3.heading Content

          transition(name="fade")
            vue-mce(
              v-show="!leaving"
              v-model="entry.content"
              :getHeight="getHeight"
            )

  //- v-toolbar(dark)
  //-   v-layout(row)
  //-     v-layout(column)
  //-       h1 {{entry.display}}
  //-       p Wello!
  //-
  //-     v-layout(column)
  //-       transition(name="zoom")
  //-         v-btn(
  //-           v-show="changed"
  //-           @click="saveEntry"
  //-           fab
  //-           color="primary"
  //-           small
  //-           :loading="saving"
  //-         )
  //-           v-icon save
  //-
  //- transition(name="fade")
  //-   vue-mce(
  //-     v-show="!leaving"
  //-     v-model="entry.content",
  //-     :getHeight="getHeight",
  //-     @input="changed = true"
  //-   )
</template>
